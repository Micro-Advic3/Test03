@action(detail=False, methods=['get'])
def explore(self, request):
    """
    Explore ALL available dashboard data with joins.
    """
    limit = int(request.query_params.get('limit', 50))
    tag_filter = request.query_params.get('tag', None)
    
    # Get all views (dashboards) with ALL related data
    queryset = View.objects.select_related(
        'workbook',
        'workbook__project',
        'workbook__site',
        'workbook__owner',
        'site'
    ).filter(
        is_deleted=False,
        workbook__is_deleted=False,
        # sheettype='dashboard'  # Commented out for now
    )
    
    # Filter by project if requested
    if tag_filter:
        queryset = queryset.filter(workbook__project__name__icontains=tag_filter)
    
    # Order by most recently updated
    queryset = queryset.order_by('-updated_at')[:limit]
    
    # Build response with ALL data
    results = []
    for view in queryset:
        # Calculate last refresh time - FIXED VERSION
        try:
            if view.updated_at:
                # Simple string format - no timezone issues
                last_refresh = view.updated_at.strftime('%Y-%m-%d')
            else:
                last_refresh = "N/A"
        except:
            last_refresh = "N/A"
        
        # Get view count from http_requests (last 30 days)
        try:
            thirty_days_ago = timezone.now() - timedelta(days=30)
            view_count = HttpRequest.objects.filter(
                currentsheet=view.name,
                created_at__gte=thirty_days_ago
            ).count()
        except:
            view_count = 0
        
        results.append({
            'id': str(view.id),
            'view_id': view.id,
            'view_name': view.name,
            'view_title': view.title or view.name,
            'view_luid': str(view.luid),
            
            # Workbook info
            'workbook_id': view.workbook.id,
            'workbook_name': view.workbook.name,
            
            # Project info (could be your "tag")
            'project_id': view.workbook.project.id,
            'project_name': view.workbook.project.name,
            
            # Site info
            'site_id': view.site.id,
            'site_name': view.site.name,
            
            # Owner info
            'owner_id': view.workbook.owner.id,
            'owner_system_user_id': view.workbook.owner.system_user_id,
            
            # Timestamps
            'created_at': view.created_at.isoformat() if view.created_at else None,
            'updated_at': view.updated_at.isoformat() if view.updated_at else None,
            'last_refresh': last_refresh,
            'first_published_at': view.first_published_at.isoformat() if view.first_published_at else None,
            
            # Metadata
            'sheettype': view.sheettype,
            'state': view.state,
            'repository_url': view.repository_url,
            
            # Calculated fields
            'view_count_30_days': view_count,
            
            # Potential Tableau URL
            'tableau_url_base': f"https://tableau.yourcompany.com/views/{view.workbook.repository_url}/{view.repository_url}",
        })
    
    return Response({
        'count': len(results),
        'results': results,
        'help': {
            'message': 'This shows ALL available data. Use this to decide what to display on Insights page!',
            'available_fields': list(results[0].keys()) if results else [],
        }
    })
