:root {  
  \--bg-primary: \#060814;  
  \--bg-surface: \#0c1126;  
  \--bg-card: \#121938;  
  \--bg-input: \#080c1c;  
  \--border-color: \#212c58;  
  \--border-highlight: \#3b4e94;  
  \--text-main: \#f8fafc;  
  \--text-muted: \#94a3b8;  
  \--text-dim: \#64748b;  
  \--accent-cyan: \#06b6d4;  
  \--accent-emerald: \#10b981;  
  \--accent-indigo: \#6366f1;  
  \--accent-amber: \#f59e0b;  
  \--accent-rose: \#f43f5e;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding: 24px;  
}

.app-container {  
  max-width: 1400px;  
  margin: 0 auto;  
}

/\* Header \*/  
.app-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  background: var(--bg-surface);  
  border: 1px solid var(--border-color);  
  padding: 20px 28px;  
  border-radius: 12px;  
  margin-bottom: 24px;  
}

.logo-group {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.logo-badge {  
  background: linear-gradient(135deg, var(--accent-amber), var(--accent-indigo));  
  color: \#020617;  
  font-weight: 900;  
  font-size: 18px;  
  padding: 8px 14px;  
  border-radius: 8px;  
}

.logo-title h1 {  
  font-size: 20px;  
  font-weight: 700;  
}

.logo-title p {  
  font-size: 12px;  
  color: var(--text-muted);  
}

.preset-controls {  
  display: flex;  
  gap: 10px;  
  align-items: center;  
}

.preset-select {  
  background: var(--bg-input);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 12px;  
  border-radius: 6px;  
  font-size: 13px;  
}

/\* Grid Layout \*/  
.grid-main {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 24px;  
  margin-bottom: 24px;  
}

@media (max-width: 1024px) {  
  .grid-main {  
    grid-template-columns: 1fr;  
  }  
}

.card {  
  background: var(--bg-surface);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 24px;  
}

.card-title {  
  font-size: 16px;  
  font-weight: 700;  
  margin-bottom: 16px;  
  display: flex;  
  align-items: center;  
  justify-content: space-between;  
  color: var(--text-main);  
}

/\* Score Banner \*/  
.score-banner {  
  display: flex;  
  align-items: center;  
  gap: 24px;  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  padding: 18px 24px;  
  border-radius: 10px;  
  margin-bottom: 20px;  
}

.grade-circle {  
  width: 64px;  
  height: 64px;  
  border-radius: 50%;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-size: 24px;  
  font-weight: 800;  
  background: var(--bg-input);  
  border: 2px solid var(--accent-emerald);  
  color: var(--accent-emerald);  
}

.grade-circle.grade-f {  
  border-color: var(--accent-rose);  
  color: var(--accent-rose);  
}

.grade-circle.grade-a {  
  border-color: var(--accent-cyan);  
  color: var(--accent-cyan);  
}

.score-stats h3 {  
  font-size: 18px;  
  font-weight: 700;  
}

.score-stats p {  
  font-size: 13px;  
  color: var(--text-muted);  
}

/\* Check Items \*/  
.checks-list {  
  display: flex;  
  flex-direction: column;  
  gap: 10px;  
  max-height: 480px;  
  overflow-y: auto;  
}

.check-item {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 12px 16px;  
  display: flex;  
  flex-direction: column;  
  gap: 4px;  
}

.check-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.check-name {  
  font-size: 13px;  
  font-weight: 600;  
}

.badge {  
  font-size: 11px;  
  font-weight: 700;  
  padding: 2px 8px;  
  border-radius: 4px;  
  text-transform: uppercase;  
}

.badge-pass { background: rgba(16, 185, 129, 0.2); color: var(--accent-emerald); }  
.badge-warn { background: rgba(245, 158, 11, 0.2); color: var(--accent-amber); }  
.badge-fail { background: rgba(244, 63, 94, 0.2); color: var(--accent-rose); }  
.badge-info { background: rgba(99, 102, 241, 0.2); color: var(--accent-indigo); }

.check-desc {  
  font-size: 12px;  
  color: var(--text-muted);  
}

.check-remediation {  
  font-size: 11px;  
  color: var(--accent-cyan);  
  margin-top: 4px;  
}

/\* Code and Textarea \*/  
textarea.code-editor {  
  width: 100%;  
  height: 280px;  
  background: var(--bg-input);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  color: var(--text-main);  
  font-family: var(--font-mono);  
  font-size: 12px;  
  padding: 12px;  
  resize: vertical;  
  line-height: 1.4;  
}

/\* Inspector Pills \*/  
.inspector-pills {  
  display: grid;  
  grid-template-columns: repeat(2, 1fr);  
  gap: 10px;  
  margin-top: 14px;  
}

.pill-card {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 6px;  
  padding: 10px 12px;  
}

.pill-label {  
  font-size: 11px;  
  color: var(--text-dim);  
  text-transform: uppercase;  
  margin-bottom: 2px;  
}

.pill-val {  
  font-size: 13px;  
  font-weight: 600;  
  font-family: var(--font-mono);  
  color: var(--text-main);  
}

/\* Simulation Section \*/  
.sim-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 20px;  
  margin-top: 16px;  
}

@media (max-width: 900px) {  
  .sim-grid {  
    grid-template-columns: 1fr;  
  }  
}

.sim-box {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 16px;  
}

.sim-timeline {  
  display: flex;  
  flex-direction: column;  
  gap: 8px;  
  margin-top: 12px;  
}

.timeline-step {  
  padding: 8px 12px;  
  border-radius: 6px;  
  background: var(--bg-input);  
  font-size: 12px;  
  border-left: 3px solid var(--accent-emerald);  
}

.timeline-step.step-outage {  
  border-left-color: var(--accent-rose);  
  background: rgba(244, 63, 94, 0.08);  
}

/\* Synthesizer Tabs \*/  
.synth-tabs {  
  display: flex;  
  gap: 8px;  
  margin-bottom: 12px;  
}

.tab-btn {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  color: var(--text-muted);  
  padding: 6px 14px;  
  border-radius: 6px;  
  font-size: 12px;  
  cursor: pointer;  
  font-weight: 600;  
}

.tab-btn.active {  
  background: var(--accent-amber);  
  color: \#020617;  
  border-color: var(--accent-amber);  
}

pre.code-block {  
  background: var(--bg-input);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 16px;  
  font-family: var(--font-mono);  
  font-size: 12px;  
  color: \#cbd5e1;  
  max-height: 340px;  
  overflow-y: auto;  
  line-height: 1.45;  
}

.btn {  
  background: linear-gradient(135deg, var(--accent-amber), var(--accent-indigo));  
  color: \#020617;  
  font-weight: 800;  
  border: none;  
  padding: 8px 16px;  
  border-radius: 6px;  
  cursor: pointer;  
  font-size: 13px;  
  transition: opacity 0.15s ease;  
}

.btn:hover {  
  opacity: 0.9;  
}

.btn-secondary {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
}  
