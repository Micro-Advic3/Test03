// src/components/features/insights/insights.tsx
"use client";

import { useState, useEffect, useRef } from "react";
import { useRouter } from "next/navigation";
import { Header } from "@/components/layout/Header";
import { ChevronLeft, ChevronRight } from "react-icons/fi";
import { ScrollToTopButton } from "@/tailwind/components/scroll-to-top-button";
import { url } from "inspector";
import TableauAPI from "@/lib/api";
import { Dashboard } from "@/lib/types";

/**
 * InsightCard Component (Inline)
 * Individual dashboard card
 */
interface CardProps {
  card: Dashboard;
  onExpand: (url: string) => void;
}

function Card({ card, onExpand }: CardProps) {
  return (
    <div className="insight-card" onClick={() => onExpand(card.tableau_url)}>
      <div className="insight-card-icon">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2}>
          <rect x="3" y="3" width="7" height="7" />
          <rect x="14" y="3" width="7" height="7" />
          <rect x="14" y="14" width="7" height="7" />
          <rect x="3" y="14" width="7" height="7" />
        </svg>
      </div>
      <div className="insight-card-content">
        <h3 className="insight-card-title">
          {card.customized_name || card.view_name || card.view_title || "Untitled"}
        </h3>
        <p className="insight-card-subtitle">{card.workbook_name}</p>
        {card.view_count !== undefined && card.view_count > 0 && (
          <span className="insight-tag">{card.view_count} views</span>
        )}
      </div>
      <div className="insight-expand-icon">
        <ChevronRight />
      </div>
    </div>
  );
}

/**
 * ScrollSection Component (Inline)
 * Netflix-style horizontal scrolling
 */
interface ScrollSectionProps {
  title: string;
  cards: Dashboard[];
  onExpand: (url: string) => void;
}

function ScrollSection({ title, cards, onExpand }: ScrollSectionProps) {
  const scrollRef = useRef<HTMLDivElement>(null);
  const sectionRef = useRef<HTMLDivElement>(null);

  const handleWheel = (e: WheelEvent) => {
    if (scrollRef.current && sectionRef.current?.contains(e.target as Node)) {
      e.preventDefault();
      scrollRef.current.scrollLeft += e.deltaY;
    }
  };

  useEffect(() => {
    const section = sectionRef.current;
    if (section) {
      section.addEventListener("wheel", handleWheel, { passive: false });
      return () => section.removeEventListener("wheel", handleWheel);
    }
  }, []);

  const scrollLeft = () => {
    if (scrollRef.current) {
      scrollRef.current.scrollBy({ left: -300, behavior: "smooth" });
    }
  };

  const scrollRight = () => {
    if (scrollRef.current) {
      scrollRef.current.scrollBy({ left: 300, behavior: "smooth" });
    }
  };

  if (cards.length === 0) return null;

  return (
    <section className="insight-section" ref={sectionRef}>
      <div className="insight-nav-btn insight-nav-left" onClick={scrollLeft}>
        <ChevronLeft />
      </div>
      
      <div className="insight-section-content">
        <h2 className="insight-scroll-title">{title}</h2>
        <div className="insight-scroll" ref={scrollRef}>
          {cards.map((card) => (
            <Card key={card.id} card={card} onExpand={onExpand} />
          ))}
        </div>
      </div>

      <button className="insight-nav-btn insight-nav-right" onClick={scrollRight}>
        <ChevronRight />
      </button>
    </section>
  );
}

/**
 * TableauModal Component (Inline)
 * Full-screen dashboard viewer
 */
interface TableauModalProps {
  url: string | null;
  onClose: () => void;
}

function TableauModal({ url, onClose }: TableauModalProps) {
  if (!url) return null;

  return (
    <div className="insight-modal" onClick={onClose}>
      <div className="insight-modal-content" onClick={(e) => e.stopPropagation()}>
        <div className="insight-modal-header">
          <h2>Dashboard</h2>
          <button onClick={onClose}>✕</button>
        </div>
        <iframe src={url} className="insight-iframe" />
      </div>
    </div>
  );
}

/**
 * Main Insights Component
 * Complete page with API integration
 */
export default function Insights() {
  const [tableauUrl, setTableauUrl] = useState<string | null>(null);

  // API Data State
  const [pinnedCards, setPinnedCards] = useState<Dashboard[]>([]);
  const [recommendedCards, setRecommendedCards] = useState<Dashboard[]>([]);
  const [permissionedCards, setPermissionedCards] = useState<Dashboard[]>([]);

  // Loading & Error
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  /**
   * Fetch dashboards from API
   */
  useEffect(() => {
    const fetchDashboards = async () => {
      try {
        setLoading(true);
        setError(null);

        // Fetch data in parallel
        const [exploreData, recentData, popularData] = await Promise.all([
          TableauAPI.explore("", 20),
          TableauAPI.getRecent(10),
          TableauAPI.getPopular(10),
        ]);

        // Map to sections
        setPinnedCards(exploreData.results.slice(0, 10));
        setRecommendedCards(popularData.popular_dashboards);
        setPermissionedCards(recentData.recent_dashboards);

      } catch (err) {
        console.error("Failed to fetch dashboards:", err);
        setError("Failed to load dashboards. Please try again.");
      } finally {
        setLoading(false);
      }
    };

    fetchDashboards();
  }, []);

  /**
   * Handle card click - open Tableau modal
   */
  const handleExpand = (url: string) => {
    setTableauUrl(url);
  };

  // Loading State
  if (loading) {
    return (
      <>
        <Header title="Insights" />
        <div className="insights-page">
          <div style={{ padding: "2rem", textAlign: "center" }}>
            <p>Loading dashboards...</p>
          </div>
        </div>
      </>
    );
  }

  // Error State
  if (error) {
    return (
      <>
        <Header title="Insights" />
        <div className="insights-page">
          <div style={{ padding: "2rem", textAlign: "center", color: "red" }}>
            <p>{error}</p>
            <button onClick={() => window.location.reload()}>Retry</button>
          </div>
        </div>
      </>
    );
  }

  return (
    <>
      <Header title="Insights" />
      <div className="insights-page">
        <ScrollSection 
          title="Pinned By Me" 
          cards={pinnedCards} 
          onExpand={handleExpand} 
        />
        <ScrollSection 
          title="Recommended" 
          cards={recommendedCards} 
          onExpand={handleExpand} 
        />
        <ScrollSection 
          title="Permissioned" 
          cards={permissionedCards} 
          onExpand={handleExpand} 
        />

        {tableauUrl && (
          <TableauModal url={tableauUrl} onClose={() => setTableauUrl(null)} />
        )}
      </div>
    </>
  );
}
