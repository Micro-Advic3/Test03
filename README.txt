@action(detail=False, methods=['get'])
def explore(self, request):
    """
    Explore ONLY accessible dashboards with known working URLs.
    Testing with: Marketing Teams Details, General CC Distribution, 
    Marketing Overview, Client Details, Custom Table, Custom Dashboard
    """
    
    # List of dashboard names we have access to
    accessible_dashboards = [
        'Marketing Teams Details',
        '2_GeneralCCDistribution',
        'Marketing Overview', 
        'Client Details',
        'Custom Table',
        'CustomDashboard'
    ]
    
    # Get views matching these names
    queryset = View.objects.select_related(
        'workbook',
        'workbook__project',
        'workbook__site',
        'workbook__owner',
        'site'
    ).filter(
        is_deleted=False,
        workbook__is_deleted=False,
        name__in=accessible_dashboards  # Only these dashboards!
    ).order_by('name')
    
    # Build response
    results = []
    for view in queryset:
        # Calculate last refresh time
        try:
            if view.updated_at:
                last_refresh = view.updated_at.strftime('%Y-%m-%d')
            else:
                last_refresh = "N/A"
        except:
            last_refresh = "N/A"
        
        # View count disabled for performance
        view_count = 0
        
        results.append({
            'id': str(view.id),
            'view_id': view.id,
            'view_index': view.index,
            'view_name': view.name,
            'view_title': view.title or view.name,
            'view_luid': str(view.luid),
            'repository_url': view.repository_url,
            
            # Workbook info
            'workbook_id': view.workbook.id,
            'workbook_name': view.workbook.name,
            'workbook_repository_url': view.workbook.repository_url,
            
            # Project info
            'project_id': view.workbook.project.id,
            'project_name': view.workbook.project.name,
            
            # Site info
            'site_id': view.site.id,
            'site_name': view.site.name,
            'site_url_namespace': view.site.url_namespace,
            
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
            
            # Calculated fields
            'view_count_30_days': view_count,
        })
    
    return Response({
        'count': len(results),
        'results': results,
        'message': f'Showing only {len(results)} accessible dashboards for testing',
        'accessible_list': accessible_dashboards
    })
