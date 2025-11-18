@action(detail=False, methods=['get'])
def explore(self, request):
    """
    COMPLETE dashboard exploration with ALL joins and FULL data.
    Returns EVERYTHING needed for URL building and display.
    """
    
    limit = int(request.query_params.get('limit', 100))
    
    # Get views with FULL joins across ALL tables
    queryset = View.objects.select_related(
        'workbook',
        'workbook__project',
        'workbook__site',
        'workbook__owner',
        'site'
    ).filter(
        is_deleted=False,
        workbook__is_deleted=False
    ).order_by('-updated_at')[:limit]
    
    # Build COMPLETE response with ALL data
    results = []
    for view in queryset:
        # Calculate last refresh time
        try:
            if view.updated_at:
                time_diff = timezone.now() - view.updated_at
                hours_ago = int(time_diff.total_seconds() / 3600)
                if hours_ago < 24:
                    last_refresh = f"{hours_ago}h ago"
                else:
                    days_ago = int(hours_ago / 24)
                    last_refresh = f"{days_ago}d ago"
            else:
                last_refresh = "N/A"
        except:
            last_refresh = "N/A"
        
        # Get view count (disabled for performance)
        view_count = 0
        
        # Build COMPLETE data object
        results.append({
            # ========== VIEW INFO ==========
            'id': str(view.id),
            'view_id': view.id,
            'view_name': view.name,
            'view_title': view.title or view.name,
            'view_luid': str(view.luid),
            'view_index': view.index,
            'repository_url': view.repository_url,
            'sheet_id': view.sheet_id if hasattr(view, 'sheet_id') else None,
            
            # ========== WORKBOOK INFO ==========
            'workbook_id': view.workbook.id,
            'workbook_name': view.workbook.name,
            'workbook_repository_url': view.workbook.repository_url,
            'workbook_luid': str(view.workbook.luid) if view.workbook.luid else None,
            
            # ========== PROJECT INFO ==========
            'project_id': view.workbook.project.id,
            'project_name': view.workbook.project.name,
            'project_description': view.workbook.project.description if hasattr(view.workbook.project, 'description') else None,
            
            # ========== SITE INFO ==========
            'site_id': view.site.id,
            'site_name': view.site.name,
            'site_url_namespace': view.site.url_namespace,
            'site_luid': str(view.site.luid) if view.site.luid else None,
            
            # ========== OWNER INFO ==========
            'owner_id': view.workbook.owner.id,
            'owner_system_user_id': view.workbook.owner.system_user_id,
            'owner_name': view.workbook.owner.name if hasattr(view.workbook.owner, 'name') else None,
            
            # ========== TIMESTAMPS ==========
            'created_at': view.created_at.isoformat() if view.created_at else None,
            'updated_at': view.updated_at.isoformat() if view.updated_at else None,
            'last_refresh': last_refresh,
            'first_published_at': view.first_published_at.isoformat() if view.first_published_at else None,
            
            # ========== METADATA ==========
            'sheettype': view.sheettype,
            'state': view.state,
            'revision': view.revision if hasattr(view, 'revision') else None,
            'description': view.description if hasattr(view, 'description') else None,
            
            # ========== ANALYTICS ==========
            'view_count_30_days': view_count,
            'view_count': view.view_count if hasattr(view, 'view_count') else 0,
            
            # ========== URL BUILDING COMPONENTS ==========
            # These are the KEY fields for building Tableau URLs!
            'url_components': {
                'base': 'https://tableau.cib.echonet/#/site',
                'site_namespace': view.site.url_namespace,
                'views_path': 'views',
                'repository_path': view.repository_url,
                'workbook_repo': view.workbook.repository_url,
                'view_name_clean': view.name,
                'index': view.index,
            },
            
            # ========== SUGGESTED TABLEAU URL ==========
            # Built URL (you can test this!)
            'tableau_url_suggestion': f"https://tableau.cib.echonet/#/site/{view.site.url_namespace}/views/{view.workbook.repository_url}/{view.name}?:iid={view.index}",
            'tableau_url_alt': f"https://tableau.cib.echonet/#/site/{view.site.url_namespace}/views/{view.repository_url.replace('/sheets/', '/')}?:iid={view.index}" if '/sheets/' in view.repository_url else None,
        })
    
    return Response({
        'count': len(results),
        'limit': limit,
        'message': f'COMPLETE data with ALL joins! Showing {len(results)} dashboards.',
        'data_includes': {
            'view_info': 'name, title, index, repository_url, luid, sheet_id',
            'workbook_info': 'name, repository_url, luid',
            'project_info': 'name, description',
            'site_info': 'name, url_namespace, luid',
            'owner_info': 'system_user_id, name',
            'timestamps': 'created, updated, last_refresh, first_published',
            'metadata': 'sheettype, state, revision, description',
            'analytics': 'view_counts',
            'url_components': 'ALL components needed to build Tableau URLs',
            'suggested_urls': 'Pre-built URL suggestions to test!'
        },
        'url_building_guide': {
            'pattern': 'https://tableau.cib.echonet/#/site/{site_url_namespace}/views/{workbook_repository_url}/{view_name}?:iid={view_index}',
            'alternative': 'https://tableau.cib.echonet/#/site/{site_url_namespace}/views/{repository_url_without_sheets}?:iid={view_index}',
            'note': 'Check tableau_url_suggestion field in each result!'
        },
        'results': results
    })
