'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { FiCalendar, FiMail, FiUser, FiGlobe, FiSun, FiMoon } from 'react-icons/fi';
import { Header } from '@/components/layout/header';
import { authApi } from '@/lib/api';
import type { User, ProfileSettings, MyProfileProps } from './my-profile.types';
import './my-profile.css';

export function MyProfile({ className = '' }: MyProfileProps) {
  const router = useRouter();
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [settings, setSettings] = useState<ProfileSettings>({
    emailNotifications: true,
    pushNotifications: false,
    darkMode: false,
    language: 'English',
  });

  // Fetch user data on mount
  useEffect(() => {
    const fetchUser = async () => {
      try {
        const userData = await authApi.getCurrentUser();
        if (userData !== null) {
          setUser(userData);
        } else {
          // User not authenticated - redirect to login
          router.push('/');
        }
      } catch (err) {
        setError('Failed to load user data');
        console.error('Error fetching user:', err);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [router]);

  // Load dark mode preference
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
      router.push('/');
    } catch (err) {
      console.error('Logout failed:', err);
    }
  };

  if (loading) {
    return (
      <>
        <Header title="My Profile" />
        <div className="flex items-center justify-center min-h-[480px]">
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

  const formatDate = (dateString: string) => {
    return new Date(dateString).toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  };

  return (
    <>
      <Header title="My Profile" />
      <div className={`profile-page ${className}`}>
        {/* Profile Header Card */}
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

        {/* Contact Information */}
        <div className="profile-card">
          <h3 className="profile-card-title">Contact Information</h3>
          <div className="profile-info-list">
            <div className="profile-info-item">
              <FiMail className="profile-icon" />
              <span className="profile-label">Email:</span>
              <span className="profile-value">{user.email}</span>
            </div>
            <div className="profile-info-item">
              <FiUser className="profile-icon" />
              <span className="profile-label">Username:</span>
              <span className="profile-value">{user.username}</span>
            </div>
            <div className="profile-info-item">
              <FiGlobe className="profile-icon" />
              <span className="profile-label">OIDC Provider:</span>
              <span className="profile-value">{user.oidc_provider || 'N/A'}</span>
            </div>
          </div>
        </div>

        {/* Preferences */}
        <div className="profile-card">
          <h3 className="profile-card-title">Preferences</h3>
          <div className="profile-setting-list">
            <div className="profile-setting-item">
              <div className="profile-setting-info">
                {settings.darkMode ? <FiMoon className="profile-icon" /> : <FiSun className="profile-icon" />}
                <span className="profile-label">Dark Mode</span>
              </div>
              <button onClick={toggleDarkMode} className={`profile-toggle ${settings.darkMode ? 'profile-toggle-active' : ''}`}>
                <span className="profile-toggle-slider"></span>
              </button>
            </div>
          </div>
        </div>

        {/* Account Details */}
        <div className="profile-card">
          <h3 className="profile-card-title">Account Details</h3>
          <div className="profile-info-list">
            <div className="profile-info-item">
              <FiCalendar className="profile-icon" />
              <span className="profile-label">Joined:</span>
              <span className="profile-value">{formatDate(user.date_joined)}</span>
            </div>
            <div className="profile-info-item">
              <FiCalendar className="profile-icon" />
              <span className="profile-label">Last Login:</span>
              <span className="profile-value">{formatDate(user.last_login)}</span>
            </div>
          </div>
        </div>

        {/* Logout Button */}
        <div className="px-6 mb-6">
          <button 
            onClick={handleLogout} 
            className="w-full min-auto bg-red-600 hover:bg-red-700 text-white font-semibold py-3 px-8 rounded-lg transition-colors"
          >
            Sign Out
          </button>
        </div>
      </div>
    </>
  );
}
