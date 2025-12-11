import logging
from django.shortcuts import redirect
from django.conf import settings

logger = logging.getLogger(__name__)

class AuthValidationMiddleware:
    """
    Nuclear option: Validate EVERY request has valid auth or kick to SSO
    """
    def __init__(self, get_response):
        self.get_response = get_response
        
    def __call__(self, request):
        # Skip for these paths
        skip_paths = [
            '/admin/',
            '/api/health/',
            '/accounts/oidc/',
            '/accounts/login/',
            '/static/',
            '/media/',
            '/_next/',
            '/favicon.ico',
        ]
        
        # If path should skip validation, continue
        if any(request.path.startswith(path) for path in skip_paths):
            return self.get_response(request)
        
        # Check if user is authenticated
        if not request.user.is_authenticated:
            logger.info(f"User not authenticated, redirecting to SSO from: {request.path}")
            # Kill any zombie session
            if request.session.session_key:
                request.session.flush()
            
            # Redirect to OIDC login
            return redirect(f'{settings.PUBLIC_DOMAIN}/accounts/oidc/bnpp-oidc/login/')
        
        # User is authenticated, but validate session has required data
        required_session_keys = ['_auth_user_id', '_auth_user_backend']
        
        if not all(key in request.session for key in required_session_keys):
            logger.warning(f"Session missing required keys for user {request.user.id}, flushing session")
            request.session.flush()
            return redirect(f'{settings.PUBLIC_DOMAIN}/accounts/oidc/bnpp-oidc/login/')
        
        # All good, continue
        response = self.get_response(request)
        return response
-------------------------------
from rest_framework import viewsets, status
from rest_framework.decorators import api_view, permission_classes, action
from rest_framework.permissions import IsAuthenticated, AllowAny
from rest_framework.response import Response
from django.contrib.auth import logout
from django.conf import settings
from .models import User
from .serializers import UserSerializer
from django.shortcuts import import redirect
from django.urls import reverse
import logging

logger = logging.getLogger(__name__)

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    ViewSet for viewing user information
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    @action(detail=False, methods=['get'])
    def me(self, request):
        """Get current user information"""
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)


@api_view(['GET'])
@permission_classes([AllowAny])
def health_check(request):
    """Health check endpoint"""
    return Response({
        'status': 'healthy',
        'service': 'qdi-portal-backend'
    })


@api_view(['POST'])
@permission_classes([IsAuthenticated])
def logout_view(request):
    """Logout endpoint"""
    logout(request)
    return Response({'message': 'Successfully logged out'})


@api_view(['GET'])
@permission_classes([AllowAny])
def oidc_callback(request):
    """
    OIDC callback endpoint
    After successful OIDC authentication, redirect to frontend
    
    This is called by django-allauth AFTER authentication
    At this point, request.user MUST be authenticated
    """
    if not request.user.is_authenticated:
        logger.error("OIDC callback called but user not authenticated")
        return Response({
            'authenticated': False,
            'error': 'Authentication failed'
        }, status=status.HTTP_401_UNAUTHORIZED)
    
    logger.info(f"OIDC callback successful for user: {request.user.email}")
    
    # User is authenticated, session is created by allauth
    # Verify session has the data we need
    if '_auth_user_id' not in request.session:
        logger.error("Session missing _auth_user_id after OIDC callback")
        return Response({
            'authenticated': False,
            'error': 'Session creation failed'
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
    
    # Redirect to frontend
    return redirect(f'{settings.PUBLIC_DOMAIN}/')


@api_view(['GET'])
@permission_classes([AllowAny])
def debug_request(request):
    """Debug endpoint to see request state"""
    from django.contrib.sites.models import Site
    site = Site.objects.get_current()
    
    return Response({
        'request_host': request.get_host(),
        'request_scheme': request.scheme,
        'meta_http_host': request.META.get('HTTP_HOST'),
        'meta_http_x_forwarded_host': request.META.get('HTTP_X_FORWARDED_HOST'),
        'site_domain': site.domain,
        'public_domain': settings.PUBLIC_DOMAIN,
        'build_absolute_uri': request.build_absolute_uri('/accounts/login/'),
    })
__________________________________
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Check for the Django Session ID cookie
  const sessionCookie = request.cookies.get('sessionid')
  
  // If no session cookie, redirect to Django OIDC login
  if (!sessionCookie) {
    const djangoLoginUrl = new URL('https://portal.qdi.dev.echonet/accounts/oidc/bnpp-oidc/login/')
    return NextResponse.redirect(djangoLoginUrl)
  }
  
  // Session cookie exists, let Django backend validate it
  return NextResponse.next()
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * 1. /api routes (if nextjs has its own api routes)
     * 2. /_next (static files)
     * 3. /static (images)
     * 4. /favicon.ico
     */
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
______________________________
'use client';

import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';
import { authApi } from '@/lib/api';

export function useRequireAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const router = useRouter();

  useEffect(() => {
    async function checkAuth() {
      try {
        // Call Django /api/auth/users/me/ to validate session
        const currentUser = await authApi.getCurrentUser();
        
        if (currentUser) {
          setUser(currentUser);
        } else {
          // No user data, redirect to SSO
          window.location.href = `${window.location.origin}/accounts/oidc/bnpp-oidc/login/`;
        }
      } catch (error: any) {
        console.error('[AUTH] Error during auth check:', error);
        // API call failed, redirect to SSO
        window.location.href = `${window.location.origin}/accounts/oidc/bnpp-oidc/login/`;
      } finally {
        setLoading = false;
      }
    }

    checkAuth();
  }, [router]);

  return { user, loading };
}
