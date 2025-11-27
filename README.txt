'use client';

import { useState, useEffect } from 'react';
import { Calendar, Mail, Briefcase, Globe, Sun, Moon } from 'react-icons/fi';
import { Header } from '@/components/layout/header';
import { authApi } from '@/lib/api';
import { formatDate } from '@/lib/utils';
import type { User, ProfileSettings, MyProfileProps } from './my-profile.types';

export function MyProfile({ className = '' }: MyProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [settings, setSettings] = useState<ProfileSettings>({
    emailNotifications: true,
    pushNotifications: false,
    darkMode: false,
    language: 'English',
  });

  useEffect(() => {
    const fetchUser = async () => {
      try {
        const userData = await authApi.getCurrentUser();
        if (userData) {
          setUser(userData);
        } else {
          setError('Not authenticated');
        }
      } catch (err) {
        setError('Failed to load user data');
      } finally {
        setLoading(false);
      }
    };
    fetchUser();
  }, []);

  useEffect(() => {
    const savedDarkMode = localStorage.getItem('darkMode');
    if (savedDarkMode) {
      const isDark = savedDarkMode === 'true';
      setSettings(prev => ({ ...prev, darkMode: isDark }));
      document.documentElement.classList.toggle('dark-mode', isDark);
    }
  }, []);

  const toggleDarkMode = () => {
    const newDarkMode = !settings.darkMode;
    setSettings(prev => ({ ...prev, darkMode: newDarkMode }));
    document.documentElement.classList.toggle('dark-mode', newDarkMode);
    localStorage.setItem('darkMode', String(newDarkMode));
  };

  const handleLogout = async () => {
    try {
      await authApi.logout();
      window.location.href = '/';
    } catch (err) {
      console.error('Logout failed:', err);
    }
  };

  if (loading) {
    return (
      <>
        <Header title="My Profile" />
        <div className="flex items-center justify-center min-h-[400px]">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-primary-600"></div>
        </div>
      </>
    );
  }

  if (error || !user) {
    return (
      <>
        <Header title="My Profile" />
        <div className="bg-red-50 border border-red-200 rounded-lg p-6 m-8">
          <p className="text-red-800">{error || 'User not found'}</p>
          <a href="/" className="text-primary-600 hover:text-primary-700 mt-2 inline-block">
            Go to login
          </a>
        </div>
      </>
    );
  }

  return (
    <>
      <Header title="My Profile" />
      <div className="profile-page">
        <div className="profile-header-card">
          <div className="profile-avatar-section">
            <div className="profile-avatar">
              {user.profile_picture ? (
                <img src={user.profile_picture} alt={user.full_name} className="profile-avatar-img" />
              ) : (
                <div className="profile-avatar-img bg-primary-300 flex items-center justify-center text-white text-4xl font-bold">
                  {user.first_name?.[0] || user.email[0].toUpperCase()}
                </div>
              )}
            </div>
            <div className="profile-info">
              <h1 className="profile-name">{user.full_name}</h1>
              <p className="profile-role">{user.oidc_provider || 'SSO User'}</p>
            </div>
          </div>
        </div>

        <div className="profile-grid">
          <div className="profile-card">
            <h3 className="profile-card-title">Contact Information</h3>
            <div className="profile-info-list">
              <div className="profile-info-item">
                <Mail className="profile-icon" />
                <span className="profile-label">Email</span>
                <span className="profile-value">{user.email}</span>
              </div>
              <div className="profile-info-item">
                <Briefcase className="profile-icon" />
                <span className="profile-label">Username</span>
                <span className="profile-value">{user.username}</span>
              </div>
              <div className="profile-info-item">
                <Globe className="profile-icon" />
                <span className="profile-label">OIDC Provider</span>
                <span className="profile-value">{user.oidc_provider || 'N/A'}</span>
              </div>
            </div>
          </div>

          <div className="profile-card">
            <h3 className="profile-card-title">Preferences</h3>
            <div className="profile-setting-list">
              <div className="profile-setting-item">
                <Calendar className="profile-icon" />
                <span className="profile-label">Joined</span>
                <span className="profile-value">{formatDate(user.date_joined)}</span>
              </div>
              <div className="profile-setting-item">
                <Calendar className="profile-icon" />
                <span className="profile-label">Last Login</span>
                <span className="profile-value">{formatDate(user.last_login)}</span>
              </div>
              <div className="profile-setting-item">
                <div className="profile-setting-info">
                  {settings.darkMode ? <Moon className="profile-icon" /> : <Sun className="profile-icon" />}
                  <span className="profile-label">Dark Mode</span>
                </div>
                <button onClick={toggleDarkMode} className={`profile-toggle ${settings.darkMode ? 'profile-toggle-active' : ''}`}>
                  <span className="profile-toggle-slider"></span>
                </button>
              </div>
            </div>
          </div>
        </div>

        <div className="px-8 mt-6">
          <button onClick={handleLogout} className="w-full md:w-auto bg-red-600 hover:bg-red-700 text-white font-semibold py-3 px-8 rounded-lg transition-colors">
            Sign Out
          </button>
        </div>
      </div>
    </>
  );
}
