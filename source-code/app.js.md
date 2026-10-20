/\*\*  
 \* CronPulse Pro \- Application Controller & Visual Orchestrator  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const cronEngine \= new CronEngine();

  // State  
  let currentAst \= null;  
  let activeCodeTab \= 'node';  
  let currentTimezone \= 'UTC';

  // Presets  
  const PRESETS \= {  
    'financial-settlement': {  
      cron: '0 0 \* \* \*',  
      jobName: 'financial\_settlement'  
    },  
    'saas-metering': {  
      cron: '\*/15 \* \* \* \*',  
      jobName: 'saas\_metering'  
    },  
    'high-freq-heartbeat': {  
      cron: '\*/30 \* \* \* \* \*',  
      jobName: 'microservice\_heartbeat'  
    }  
  };

  // DOM Elements  
  const cronInput \= document.getElementById('cron-input');  
  const syntaxErrorMsg \= document.getElementById('syntax-error-msg');  
  const explanationText \= document.getElementById('explanation-text');  
  const formatBadge \= document.getElementById('format-badge');  
  const fieldsBreakdown \= document.getElementById('fields-breakdown');  
  const selectTimezone \= document.getElementById('select-timezone');  
  const scheduleTableBody \= document.getElementById('schedule-table-body');  
  const inputJobName \= document.getElementById('input-job-name');  
  const inputLockTtl \= document.getElementById('input-lock-ttl');

  // Code Tab DOM  
  const codeSnippetDisplay \= document.getElementById('code-snippet-display');  
  const tabBtns \= document.querySelectorAll('.tab-btn');  
  const btnCopyCode \= document.getElementById('btn-copy-code');  
  const btnDownloadSnippet \= document.getElementById('btn-download-snippet');  
  const presetBtns \= document.querySelectorAll('.header-actions .btn-preset');  
  const quickCadenceBtns \= document.querySelectorAll('\[data-cron\]');

  function updateUi() {  
    const rawExpr \= cronInput.value.trim();  
    currentAst \= cronEngine.parse(rawExpr);

    if (\!currentAst.valid) {  
      syntaxErrorMsg.style.display \= 'block';  
      syntaxErrorMsg.textContent \= \`Syntax Error: \${currentAst.error}\`;  
      explanationText.textContent \= 'Invalid expression. Check syntax.';  
      formatBadge.textContent \= 'INVALID CRON';  
      formatBadge.style.color \= '\#ef4444';  
      scheduleTableBody.innerHTML \= '\<tr\>\<td colspan="4" style="text-align: center; color: \#9ca3af; padding: 20px;"\>Fix expression errors to view execution schedule.\</td\>\</tr\>';  
      return;  
    }

    syntaxErrorMsg.style.display \= 'none';  
    formatBadge.style.color \= 'var(--accent-cyan)';  
    formatBadge.textContent \= currentAst.format.toUpperCase().replace(/\_/g, ' ');

    // 1\. Natural Language Explanation  
    explanationText.textContent \= cronEngine.explain(currentAst);

    // 2\. Update Field Pills  
    renderFieldPills(currentAst);

    // 3\. Compute Next 10 Executions  
    renderScheduleTable(currentAst);

    // 4\. Update Code Synthesizer  
    updateCodeDisplay();  
  }

  function renderFieldPills(ast) {  
    fieldsBreakdown.innerHTML \= '';  
    const fields \= ast.fields;  
    const is6Field \= ast.format \=== 'quartz\_6\_field';  
    const is7Field \= ast.format \=== 'enterprise\_7\_field';

    if (is6Field || is7Field) {  
      fieldsBreakdown.className \= 'fields-breakdown six-field';  
      addFieldPill('Second', fields.seconds.raw);  
    } else {  
      fieldsBreakdown.className \= 'fields-breakdown';  
    }

    addFieldPill('Minute', fields.minutes.raw);  
    addFieldPill('Hour', fields.hours.raw);  
    addFieldPill('Day (Mo)', fields.dayOfMonth.raw);  
    addFieldPill('Month', fields.month.raw);  
    addFieldPill('Day (Wk)', fields.dayOfWeek.raw);

    if (is7Field) {  
      addFieldPill('Year', fields.year.raw);  
    }  
  }

  function addFieldPill(label, val) {  
    const div \= document.createElement('div');  
    div.className \= 'field-pill';  
    div.innerHTML \= \`  
      \<span class="field-label"\>\${label}\</span\>  
      \<span class="field-val"\>\${val}\</span\>  
    \`;  
    fieldsBreakdown.appendChild(div);  
  }

  function renderScheduleTable(ast) {  
    scheduleTableBody.innerHTML \= '';  
    const now \= new Date();  
    const executions \= cronEngine.getNextExecutions(ast, 10, now);

    if (executions.length \=== 0\) {  
      scheduleTableBody.innerHTML \= '\<tr\>\<td colspan="4" style="text-align: center; color: \#9ca3af; padding: 20px;"\>No upcoming execution dates found within search window.\</td\>\</tr\>';  
      return;  
    }

    let previousDate \= now;

    executions.forEach((execDate, index) \=\> {  
      const row \= document.createElement('tr');  
      const formattedDate \= formatInTimezone(execDate, currentTimezone);

      // Delta from previous run  
      const deltaMs \= index \=== 0 ? execDate.getTime() \- now.getTime() : execDate.getTime() \- previousDate.getTime();  
      const deltaStr \= formatDeltaDuration(deltaMs);

      // Relative Countdown from now  
      const countdownMs \= execDate.getTime() \- now.getTime();  
      const countdownStr \= formatCountdown(countdownMs);

      row.innerHTML \= \`  
        \<td style="color: var(--accent-cyan); font-weight: bold;"\>\#\${index \+ 1}\</td\>  
        \<td style="color: \#f3f4f6; font-weight: 500;"\>\${formattedDate}\</td\>  
        \<td style="color: var(--text-muted);"\>\${index \=== 0 ? \`Starts \+\${deltaStr}\` : \`+\${deltaStr}\`}\</td\>  
        \<td style="text-align: right;"\>\<span class="badge-countdown"\>\${countdownStr}\</span\>\</td\>  
      \`;  
      scheduleTableBody.appendChild(row);  
      previousDate \= execDate;  
    });  
  }

  function formatInTimezone(date, tz) {  
    const options \= {  
      year: 'numeric',  
      month: 'short',  
      day: '2-digit',  
      hour: '2-digit',  
      minute: '2-digit',  
      second: '2-digit',  
      hour12: false  
    };

    if (tz \=== 'LOCAL') {  
      return date.toLocaleString('en-US', options) \+ ' (Local)';  
    }

    options.timeZone \= tz;  
    return date.toLocaleString('en-US', options) \+ \` (\${tz.split('/')\[1\] || tz})\`;  
  }

  function formatDeltaDuration(ms) {  
    const totalSecs \= Math.floor(ms / 1000);  
    const days \= Math.floor(totalSecs / 86400);  
    const hrs \= Math.floor((totalSecs % 86400\) / 3600);  
    const mins \= Math.floor((totalSecs % 3600\) / 60);  
    const secs \= totalSecs % 60;

    if (days \> 0\) return \`\${days}d \${hrs}h\`;  
    if (hrs \> 0\) return \`\${hrs}h \${mins}m\`;  
    if (mins \> 0\) return \`\${mins}m \${secs}s\`;  
    return \`\${secs}s\`;  
  }

  function formatCountdown(ms) {  
    const totalSecs \= Math.max(0, Math.floor(ms / 1000));  
    const hrs \= Math.floor(totalSecs / 3600);  
    const mins \= Math.floor((totalSecs % 3600\) / 60);  
    const secs \= totalSecs % 60;

    if (hrs \> 24\) {  
      const days \= Math.floor(hrs / 24);  
      return \`in \${days}d \${hrs % 24}h\`;  
    }  
    if (hrs \> 0\) return \`in \${hrs}h \${mins}m\`;  
    if (mins \> 0\) return \`in \${mins}m \${secs}s\`;  
    return \`in \${secs}s\`;  
  }

  function updateCodeDisplay() {  
    const cronExpr \= cronInput.value.trim();  
    const jobName \= inputJobName.value.trim() || 'scheduled\_task';  
    let code \= '';

    switch (activeCodeTab) {  
      case 'node':  
        code \= LockEngine.generateNodeRedisLock(cronExpr, jobName);  
        break;  
      case 'python':  
        code \= LockEngine.generatePythonRedisLock(cronExpr, jobName);  
        break;  
      case 'postgres':  
        code \= LockEngine.generatePostgreSqlLock(cronExpr, 987654);  
        break;  
      case 'k8s':  
        code \= LockEngine.generateK8sCronJobYaml(cronExpr, jobName);  
        break;  
      case 'golang':  
        code \= LockEngine.generateGoLock(cronExpr, jobName);  
        break;  
      default:  
        code \= LockEngine.generateNodeRedisLock(cronExpr, jobName);  
    }

    codeSnippetDisplay.textContent \= code;  
  }

  // Event Listeners  
  cronInput.addEventListener('input', updateUi);  
  inputJobName.addEventListener('input', updateCodeDisplay);  
  inputLockTtl.addEventListener('input', updateCodeDisplay);

  selectTimezone.addEventListener('change', (e) \=\> {  
    currentTimezone \= e.target.value;  
    if (currentAst && currentAst.valid) {  
      renderScheduleTable(currentAst);  
    }  
  });

  // Preset Buttons (Header)  
  presetBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      presetBtns.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      const pKey \= btn.getAttribute('data-preset');  
      const p \= PRESETS\[pKey\];  
      if (p) {  
        cronInput.value \= p.cron;  
        inputJobName.value \= p.jobName;  
        updateUi();  
      }  
    });  
  });

  // Quick Cadence Buttons  
  quickCadenceBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      const expr \= btn.getAttribute('data-cron');  
      if (expr) {  
        cronInput.value \= expr;  
        updateUi();  
      }  
    });  
  });

  // Tab Navigation for Codegen  
  tabBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabBtns.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeCodeTab \= btn.getAttribute('data-tab');  
      updateCodeDisplay();  
    });  
  });

  // Copy Code  
  btnCopyCode.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(codeSnippetDisplay.textContent).then(() \=\> {  
      const orig \= btnCopyCode.innerHTML;  
      btnCopyCode.innerHTML \= \`\<svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"\>\<polyline points="20 6 9 17 4 12"/\>\</svg\> Copied\!\`;  
      setTimeout(() \=\> btnCopyCode.innerHTML \= orig, 2000);  
    });  
  });

  // Download Code  
  btnDownloadSnippet.addEventListener('click', () \=\> {  
    let filename \= 'cron\_worker.js';  
    if (activeCodeTab \=== 'python') filename \= 'cron\_worker.py';  
    else if (activeCodeTab \=== 'postgres') filename \= 'advisory\_lock.sql';  
    else if (activeCodeTab \=== 'k8s') filename \= 'cronjob.yaml';  
    else if (activeCodeTab \=== 'golang') filename \= 'cron\_worker.go';

    const blob \= new Blob(\[codeSnippetDisplay.textContent\], { type: 'text/plain' });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= filename;  
    a.click();  
    URL.revokeObjectURL(url);  
  });

  // Initial Load  
  updateUi();  
});  
