import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // STEP 1: Clear all old cookies FIRST (automatic cleanup)
  const response = NextResponse.next()
  
  // List of cookies that might conflict
  const cookiesToClear = [
    'sessionid',
    'csrftoken',
    'messages',
    'SMSESSION',
    'SMIDENTITY', 
    'SMAGENTNAME',
  ]
  
  // Delete each cookie
  cookiesToClear.forEach(cookieName => {
    if (request.cookies.has(cookieName)) {
      response.cookies.delete(cookieName)
    }
  })
  
  // STEP 2: Your existing SSO check logic (unchanged)
  // Check for the Django Session ID cookie
  // Default name is usually 'sessionid', check your settings.py
  const sessionCookie = request.cookies.get('sessionid')
  
  // If the cookie exists, we assume they are logged in.
  // (Verification happens when the API is actually called).
  if (sessionCookie) {
    return response  // Return response with cleared cookies
  }
  
  // If NOT LOGGED IN:
  // Redirect to the Django Allauth OIDC trigger URL.
  // Replace 'oidc' with your specific provider ID (e.g., google, keycloak, auth0)
  const djangoLoginUrl = new URL('https://portal.qdi.dev.echonet/accounts/oidc/bnpp-oidc/login/')
  
  // Optional: Add a 'next' parameter so Django knows where to return (if you have custom logic)
  // djangoLoginUrl.searchParams.set('next', request.nextUrl.pathname)
  
  return NextResponse.redirect(djangoLoginUrl)
}

export const config = {
  // Configure which paths this middleware applies to
  matcher: [
    /*
     * Match all request paths except for:
     * 1. /api routes (if nextjs has its own api routes)
     * 2. /_next (static files)
     * 3. /static (images)
     * 4. /favicon.ico
     */
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
