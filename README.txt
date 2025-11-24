"""
Database router for Tableau app
Routes all Tableau model queries to the 'tableau' database
"""


class TableauRouter:
    """
    A router to direct Tableau models to the tableau database
    """
    
    route_app_labels = {'tableau'}
    
    def db_for_read(self, model, **hints):
        """
        Attempts to read tableau models go to tableau database
        """
        if model._meta.app_label in self.route_app_labels:
            return 'tableau'
        return None
    
    def db_for_write(self, model, **hints):
        """
        Attempts to write tableau models go to tableau database
        """
        if model._meta.app_label in self.route_app_labels:
            return 'tableau'
        return None
    
    def allow_relation(self, obj1, obj2, **hints):
        """
        Allow relations if both models are in the tableau app
        """
        if (
            obj1._meta.app_label in self.route_app_labels or
            obj2._meta.app_label in self.route_app_labels
        ):
            return True
        return None
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        Make sure the tableau app only appears in the 'tableau' database
        Never run migrations on tableau database
        """
        if app_label in self.route_app_labels:
            return False  # Never migrate Tableau tables
        return None
