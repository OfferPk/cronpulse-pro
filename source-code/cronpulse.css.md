/\* CronPulse Pro \- Modern Dark Studio UI \*/  
:root {  
  \--bg-primary: \#0a0e17;  
  \--bg-secondary: \#111827;  
  \--bg-card: rgba(17, 24, 39, 0.75);  
  \--border-color: rgba(255, 255, 255, 0.1);  
  \--border-focus: \#6366f1;  
  \--text-main: \#f9fafb;  
  \--text-muted: \#9ca3af;  
  \--accent-indigo: \#6366f1;  
  \--accent-cyan: \#06b6d4;  
  \--accent-green: \#10b981;  
  \--accent-red: \#ef4444;  
  \--accent-amber: \#f59e0b;  
  \--font-sans: 'Inter', \-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;  
  \--font-mono: 'JetBrains Mono', 'Fira Code', Menlo, Monaco, Consolas, monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  font-family: var(--font-sans);  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  line-height: 1.5;  
  min-height: 100vh;  
  padding: 24px;  
}

.studio-container {  
  max-width: 1440px;  
  margin: 0 auto;  
  display: flex;  
  flex-direction: column;  
  gap: 24px;  
}

/\* Header \*/  
.studio-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding: 20px 24px;  
  background: var(--bg-card);  
  backdrop-filter: blur(16px);  
  border: 1px solid var(--border-color);  
  border-radius: 16px;  
  flex-wrap: wrap;  
  gap: 16px;  
}

.brand-wrapper {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.brand-icon {  
  width: 44px;  
  height: 44px;  
  background: linear-gradient(135deg, \#6366f1, \#06b6d4);  
  border-radius: 12px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  color: white;  
  box-shadow: 0 4px 14px rgba(99, 102, 241, 0.4);  
}

.brand-title {  
  font-size: 1.35rem;  
  font-weight: 700;  
  letter-spacing: \-0.02em;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

.version-badge {  
  font-size: 0.75rem;  
  padding: 2px 8px;  
  border-radius: 9999px;  
  background: rgba(99, 102, 241, 0.2);  
  color: \#a5b4fc;  
  border: 1px solid rgba(99, 102, 241, 0.4);  
}

.brand-subtitle {  
  font-size: 0.85rem;  
  color: var(--text-muted);  
}

.header-actions {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
  flex-wrap: wrap;  
}

.preset-pill-group {  
  display: flex;  
  gap: 8px;  
  background: rgba(0, 0, 0, 0.3);  
  padding: 4px;  
  border-radius: 10px;  
  border: 1px solid var(--border-color);  
}

.btn-preset {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 0.8rem;  
  font-weight: 500;  
  padding: 6px 12px;  
  border-radius: 6px;  
  cursor: pointer;  
  transition: all 0.2s ease;  
}

.btn-preset:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.05);  
}

.btn-preset.active {  
  color: \#fff;  
  background: var(--accent-indigo);  
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.4);  
}

/\* Explanation Hero Card \*/  
.explanation-hero {  
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.15), rgba(6, 182, 212, 0.15));  
  border: 1px solid rgba(99, 102, 241, 0.35);  
  border-radius: 16px;  
  padding: 20px 24px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  flex-wrap: wrap;  
  gap: 16px;  
}

.explanation-text {  
  font-size: 1.15rem;  
  font-weight: 600;  
  color: \#f3f4f6;  
  letter-spacing: \-0.01em;  
}

.format-pill {  
  font-family: var(--font-mono);  
  font-size: 0.78rem;  
  padding: 4px 10px;  
  background: rgba(0, 0, 0, 0.4);  
  border-radius: 6px;  
  border: 1px solid rgba(255, 255, 255, 0.15);  
  color: var(--accent-cyan);  
}

/\* Studio Layout \*/  
.studio-grid {  
  display: grid;  
  grid-template-columns: 460px 1fr;  
  gap: 24px;  
}

@media (max-width: 1024px) {  
  .studio-grid {  
    grid-template-columns: 1fr;  
  }  
}

.studio-panel {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 16px;  
  padding: 24px;  
  display: flex;  
  flex-direction: column;  
  gap: 20px;  
  backdrop-filter: blur(12px);  
}

.panel-title {  
  font-size: 1.1rem;  
  font-weight: 600;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
  padding-bottom: 12px;  
  border-bottom: 1px solid var(--border-color);  
}

/\* Cron Expression Input Area \*/  
.cron-input-box {  
  background: rgba(0, 0, 0, 0.5);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 16px;  
  display: flex;  
  flex-direction: column;  
  gap: 12px;  
}

.cron-input {  
  background: transparent;  
  border: none;  
  font-family: var(--font-mono);  
  font-size: 1.5rem;  
  font-weight: 700;  
  color: \#38bdf8;  
  letter-spacing: 0.08em;  
  width: 100%;  
  outline: none;  
}

.fields-breakdown {  
  display: grid;  
  grid-template-columns: repeat(5, 1fr);  
  gap: 6px;  
  text-align: center;  
}

.fields-breakdown.six-field {  
  grid-template-columns: repeat(6, 1fr);  
}

.field-pill {  
  background: rgba(255, 255, 255, 0.04);  
  border: 1px solid var(--border-color);  
  border-radius: 6px;  
  padding: 6px 4px;  
  display: flex;  
  flex-direction: column;  
  gap: 2px;  
}

.field-label {  
  font-size: 0.68rem;  
  color: var(--text-muted);  
  text-transform: uppercase;  
}

.field-val {  
  font-family: var(--font-mono);  
  font-size: 0.85rem;  
  font-weight: 600;  
  color: var(--text-main);  
}

/\* Timezone Switcher \*/  
.tz-switcher-wrapper {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  flex-wrap: wrap;  
  gap: 8px;  
}

.form-select {  
  background: rgba(0, 0, 0, 0.4);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 12px;  
  border-radius: 8px;  
  font-size: 0.85rem;  
  font-family: inherit;  
  outline: none;  
}

.form-select:focus {  
  border-color: var(--border-focus);  
}

/\* Execution Schedule Table \*/  
.schedule-table {  
  width: 100%;  
  border-collapse: collapse;  
  font-family: var(--font-mono);  
  font-size: 0.82rem;  
}

.schedule-table th {  
  text-align: left;  
  padding: 10px 12px;  
  color: var(--text-muted);  
  font-size: 0.72rem;  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
  border-bottom: 1px solid var(--border-color);  
}

.schedule-table td {  
  padding: 10px 12px;  
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);  
}

.badge-countdown {  
  background: rgba(16, 185, 129, 0.15);  
  color: \#34d399;  
  border: 1px solid rgba(16, 185, 129, 0.3);  
  padding: 2px 8px;  
  border-radius: 4px;  
  font-size: 0.75rem;  
  font-weight: 600;  
}

/\* Code Synthesizer Tabs \*/  
.tabs-container {  
  display: flex;  
  flex-direction: column;  
  gap: 12px;  
}

.tab-nav {  
  display: flex;  
  gap: 6px;  
  background: rgba(0, 0, 0, 0.3);  
  padding: 4px;  
  border-radius: 10px;  
  border: 1px solid var(--border-color);  
  overflow-x: auto;  
}

.tab-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 0.82rem;  
  font-weight: 500;  
  padding: 8px 14px;  
  border-radius: 6px;  
  cursor: pointer;  
  white-space: nowrap;  
  transition: all 0.2s ease;  
}

.tab-btn:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.05);  
}

.tab-btn.active {  
  color: \#fff;  
  background: rgba(255, 255, 255, 0.12);  
  border: 1px solid rgba(255, 255, 255, 0.15);  
}

.code-display {  
  background: \#0d1117;  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 16px;  
  font-family: var(--font-mono);  
  font-size: 0.82rem;  
  color: \#c9d1d9;  
  overflow-x: auto;  
  max-height: 380px;  
  white-space: pre;  
  line-height: 1.45;  
}

.code-actions {  
  display: flex;  
  justify-content: flex-end;  
  gap: 10px;  
}

.btn {  
  display: inline-flex;  
  align-items: center;  
  justify-content: center;  
  gap: 8px;  
  padding: 10px 16px;  
  font-size: 0.88rem;  
  font-weight: 600;  
  border-radius: 8px;  
  cursor: pointer;  
  border: 1px solid transparent;  
  transition: all 0.2s ease;  
}

.btn-primary {  
  background: linear-gradient(135deg, \#6366f1, \#4f46e5);  
  color: white;  
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);  
}

.btn-primary:hover {  
  filter: brightness(1.1);  
  transform: translateY(-1px);  
}

.btn-secondary {  
  background: rgba(255, 255, 255, 0.08);  
  color: var(--text-main);  
  border: 1px solid var(--border-color);  
}

.btn-secondary:hover {  
  background: rgba(255, 255, 255, 0.12);  
}

/\* Footer \*/  
.studio-footer {  
  text-align: center;  
  font-size: 0.8rem;  
  color: var(--text-muted);  
  padding: 16px;  
  border-top: 1px solid var(--border-color);  
}  
