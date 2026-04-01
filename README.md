<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AFS SOC - Vulnerability Management Report</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Syne:wght@400;600;700;800&display=swap');

  :root {
    --bg:        #0b0e14;
    --surface:   #111620;
    --border:    #1e2535;
    --text:      #c9d1e0;
    --muted:     #5a6580;
    --accent:    #00c8ff;

    --critical:  #ff3b5c;
    --high:      #ff7c2a;
    --medium:    #f5c518;
    --low:       #4ade80;
    --info:      #60a5fa;

    --critical-bg: rgba(255,59,92,0.1);
    --high-bg:     rgba(255,124,42,0.1);
    --medium-bg:   rgba(245,197,24,0.08);
    --low-bg:      rgba(74,222,128,0.08);
    --info-bg:     rgba(96,165,250,0.08);
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Syne', sans-serif;
    font-size: 15px;
    line-height: 1.7;
    padding: 2rem 1rem 4rem;
  }

  .container { max-width: 900px; margin: 0 auto; }

  /* ── Header ─────────────────────────────── */
  header {
    border-left: 3px solid var(--accent);
    padding: 1.5rem 0 1.5rem 1.5rem;
    margin-bottom: 2.5rem;
    position: relative;
  }
  header::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(90deg, rgba(0,200,255,0.05) 0%, transparent 60%);
    pointer-events: none;
  }
  .org-tag {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.72rem;
    color: var(--accent);
    letter-spacing: 0.2em;
    text-transform: uppercase;
    margin-bottom: 0.4rem;
  }
  h1 {
    font-size: clamp(1.5rem, 4vw, 2.2rem);
    font-weight: 800;
    color: #fff;
    line-height: 1.2;
    margin-bottom: 0.75rem;
  }
  .meta-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem 1.5rem;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.75rem;
    color: var(--muted);
  }
  .meta-grid span { white-space: nowrap; }
  .meta-grid strong { color: var(--text); }

  /* ── Sections ────────────────────────────── */
  section { margin-bottom: 2.5rem; }

  h2 {
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: var(--accent);
    font-family: 'Share Tech Mono', monospace;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--border);
  }

  h3 {
    font-size: 0.95rem;
    font-weight: 700;
    color: #e2e8f0;
    margin: 1.25rem 0 0.5rem;
  }

  p { color: var(--text); margin-bottom: 0.75rem; }

  /* ── Severity badges ─────────────────────── */
  .badge {
    display: inline-block;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    padding: 0.18em 0.6em;
    border-radius: 3px;
    text-transform: uppercase;
  }
  .badge-critical { background: var(--critical-bg); color: var(--critical); border: 1px solid rgba(255,59,92,0.3); }
  .badge-high     { background: var(--high-bg);     color: var(--high);     border: 1px solid rgba(255,124,42,0.3); }
  .badge-medium   { background: var(--medium-bg);   color: var(--medium);   border: 1px solid rgba(245,197,24,0.25); }
  .badge-low      { background: var(--low-bg);      color: var(--low);      border: 1px solid rgba(74,222,128,0.25); }
  .badge-info     { background: var(--info-bg);     color: var(--info);     border: 1px solid rgba(96,165,250,0.25); }

  /* ── Tables ──────────────────────────────── */
  .table-wrap { overflow-x: auto; border-radius: 6px; border: 1px solid var(--border); margin-bottom: 1rem; }
  table { width: 100%; border-collapse: collapse; font-size: 0.875rem; }
  thead tr { background: #141926; }
  thead th {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--muted);
    padding: 0.7rem 1rem;
    text-align: left;
    border-bottom: 1px solid var(--border);
  }
  tbody tr { border-bottom: 1px solid var(--border); transition: background 0.15s; }
  tbody tr:last-child { border-bottom: none; }
  tbody tr:hover { background: rgba(255,255,255,0.03); }
  td { padding: 0.65rem 1rem; vertical-align: middle; }

  /* severity rows in findings table */
  .row-critical td:first-child { border-left: 2px solid var(--critical); }
  .row-high     td:first-child { border-left: 2px solid var(--high); }
  .row-medium   td:first-child { border-left: 2px solid var(--medium); }
  .row-low      td:first-child { border-left: 2px solid var(--low); }
  .row-info     td:first-child { border-left: 2px solid var(--info); }

  /* totals row */
  .totals-row td { font-weight: 700; color: #fff; background: #141926; }

  /* ── Scorecard ───────────────────────────── */
  .scorecard {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
    gap: 0.75rem;
    margin-bottom: 1.5rem;
  }
  .score-card {
    border-radius: 6px;
    padding: 1rem;
    text-align: center;
    border: 1px solid;
  }
  .score-card .num {
    display: block;
    font-size: 2rem;
    font-weight: 800;
    line-height: 1;
    margin-bottom: 0.3rem;
  }
  .score-card .label {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }
  .sc-critical { background: var(--critical-bg); border-color: rgba(255,59,92,0.3);  }
  .sc-critical .num { color: var(--critical); }
  .sc-critical .label { color: rgba(255,59,92,0.7); }
  .sc-high     { background: var(--high-bg);     border-color: rgba(255,124,42,0.3); }
  .sc-high .num { color: var(--high); }
  .sc-high .label { color: rgba(255,124,42,0.7); }
  .sc-medium   { background: var(--medium-bg);   border-color: rgba(245,197,24,0.25); }
  .sc-medium .num { color: var(--medium); }
  .sc-medium .label { color: rgba(245,197,24,0.6); }
  .sc-low      { background: var(--low-bg);      border-color: rgba(74,222,128,0.25); }
  .sc-low .num { color: var(--low); }
  .sc-low .label { color: rgba(74,222,128,0.6); }
  .sc-info     { background: var(--info-bg);     border-color: rgba(96,165,250,0.25); }
  .sc-info .num { color: var(--info); }
  .sc-info .label { color: rgba(96,165,250,0.6); }
  .sc-total    { background: rgba(0,200,255,0.07); border-color: rgba(0,200,255,0.25); }
  .sc-total .num { color: var(--accent); }
  .sc-total .label { color: rgba(0,200,255,0.6); }

  /* ── Alert boxes ─────────────────────────── */
  .alert {
    border-left: 3px solid;
    padding: 0.85rem 1rem;
    border-radius: 0 6px 6px 0;
    margin-bottom: 0.75rem;
    font-size: 0.875rem;
  }
  .alert-critical { border-color: var(--critical); background: var(--critical-bg); }
  .alert-high     { border-color: var(--high);     background: var(--high-bg); }
  .alert strong { color: #fff; }

  /* ── Steps ───────────────────────────────── */
  .steps { counter-reset: step; list-style: none; }
  .steps li {
    counter-increment: step;
    display: flex;
    gap: 0.9rem;
    margin-bottom: 0.6rem;
    align-items: flex-start;
    font-size: 0.875rem;
  }
  .steps li::before {
    content: counter(step);
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.7rem;
    background: var(--surface);
    border: 1px solid var(--border);
    color: var(--accent);
    min-width: 1.6rem;
    height: 1.6rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 3px;
    flex-shrink: 0;
    margin-top: 0.15rem;
  }

  /* ── Code / mono ─────────────────────────── */
  code {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.8em;
    background: rgba(255,255,255,0.07);
    border: 1px solid var(--border);
    padding: 0.1em 0.4em;
    border-radius: 3px;
    color: #e2e8f0;
  }
  pre {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 1rem;
    overflow-x: auto;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.78rem;
    color: #94a3b8;
    line-height: 1.6;
    margin-bottom: 1rem;
  }

  /* ── Bullet list ─────────────────────────── */
  ul.standard { padding-left: 1.2rem; margin-bottom: 0.75rem; }
  ul.standard li { margin-bottom: 0.3rem; font-size: 0.875rem; }

  /* ── Disclaimer ──────────────────────────── */
  .disclaimer {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 1rem 1.2rem;
    font-size: 0.8rem;
    color: var(--muted);
    line-height: 1.6;
  }

  /* ── Divider ─────────────────────────────── */
  hr { border: none; border-top: 1px solid var(--border); margin: 2rem 0; }
</style>
</head>
<body>
<div class="container">

  <!-- HEADER -->
  <header>
    <div class="org-tag">Aureon Financial Systems &mdash; Security Operations Centre</div>
    <h1>Vulnerability Management &amp;<br>Security Risk Analytics</h1>
    <div class="meta-grid">
      <span><strong>Ref:</strong> AFS-SOC-VULN-2026-001</span>
      <span><strong>Period:</strong> 20&ndash;26 Feb 2026</span>
      <span><strong>Classification:</strong> Internal Use Only</span>
      <span><strong>Analyst:</strong> Emmanuel Ateji</span>
    </div>
  </header>

  <!-- OVERVIEW -->
  <section>
    <h2>Overview</h2>
    <p>This repository contains the complete deliverables for the <strong>Vulnerability Management &amp; Security Risk Analytics</strong> project conducted by the SOC of Aureon Financial Systems (AFS).</p>
    <p>The assessment covered two representative target systems using seven scan configurations across five industry-standard tools, identifying and prioritising <strong>119 unique vulnerabilities</strong> across a legacy Linux host and a customer-facing web application. The goal was to transition AFS from a reactive security posture to a structured, risk-based vulnerability management framework.</p>
  </section>

  <!-- TARGETS -->
  <section>
    <h2>Targets Assessed</h2>
    <div class="table-wrap">
      <table>
        <thead>
          <tr><th>Target</th><th>Identifier</th><th>Type</th><th>Notes</th></tr>
        </thead>
        <tbody>
          <tr>
            <td><strong>Metasploitable 2</strong></td>
            <td><code>10.0.2.4</code></td>
            <td>Linux Host</td>
            <td>Ubuntu 8.04 (EOL) &ndash; represents AFS legacy server infrastructure</td>
          </tr>
          <tr>
            <td><strong>VulnWeb PHP App</strong></td>
            <td><code>testphp.vulnweb.com</code></td>
            <td>Web Application</td>
            <td>PHP 5.6.40 &ndash; represents a customer-facing PHP service</td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>

  <!-- KEY FINDINGS -->
  <section>
    <h2>Key Findings</h2>

    <div class="scorecard">
      <div class="score-card sc-critical"><span class="num">21</span><span class="label">Critical</span></div>
      <div class="score-card sc-high">    <span class="num">36</span><span class="label">High</span></div>
      <div class="score-card sc-medium">  <span class="num">38</span><span class="label">Medium</span></div>
      <div class="score-card sc-low">     <span class="num">13</span><span class="label">Low</span></div>
      <div class="score-card sc-info">    <span class="num">11</span><span class="label">Info</span></div>
      <div class="score-card sc-total">   <span class="num">119</span><span class="label">Total</span></div>
    </div>

    <div class="table-wrap">
      <table>
        <thead>
          <tr><th>Severity</th><th>Metasploitable 2</th><th>VulnWeb App</th><th>Combined Total</th></tr>
        </thead>
        <tbody>
          <tr class="row-critical">
            <td><span class="badge badge-critical">Critical</span></td>
            <td>19</td><td>2</td><td><strong>21</strong></td>
          </tr>
          <tr class="row-high">
            <td><span class="badge badge-high">High</span></td>
            <td>33</td><td>3</td><td><strong>36</strong></td>
          </tr>
          <tr class="row-medium">
            <td><span class="badge badge-medium">Medium</span></td>
            <td>34</td><td>4</td><td><strong>38</strong></td>
          </tr>
          <tr class="row-low">
            <td><span class="badge badge-low">Low</span></td>
            <td>9</td><td>4</td><td><strong>13</strong></td>
          </tr>
          <tr class="row-info">
            <td><span class="badge badge-info">Informational</span></td>
            <td>6</td><td>5</td><td><strong>11</strong></td>
          </tr>
          <tr class="totals-row">
            <td>Total</td><td>101</td><td>18</td><td>119</td>
          </tr>
        </tbody>
      </table>
    </div>

    <h3>Critical Highlights</h3>
    <div class="alert alert-critical">
      <strong>Metasploitable 2</strong> hosts <strong>four confirmed active backdoors</strong> granting immediate root-level access with zero authentication &mdash; Ingreslock, vsftpd 2.3.4, UnrealIRCd, and ProFTPD 1.3.3c &mdash; plus a second exposed Apache Tomcat Manager on port 8180.
    </div>
    <div class="alert alert-high">
      <strong>VulnWeb App</strong> exposes SQL injection across six endpoints and transmits all credentials in cleartext over HTTP.
    </div>
    <p style="font-size:0.875rem;">Credentialed (SSH) scanning revealed <strong>383+ CVEs</strong> on Metasploitable 2 that unauthenticated scanning alone could not detect.</p>
  </section>

  <!-- TOOLS -->
  <section>
    <h2>Tools Used</h2>
    <div class="table-wrap">
      <table>
        <thead>
          <tr><th>Tool</th><th>Version</th><th>Scan Type</th></tr>
        </thead>
        <tbody>
          <tr><td><strong>Tenable Nessus</strong></td><td>Latest</td><td>Basic unauthenticated + Advanced SSH-credentialed scan</td></tr>
          <tr><td><strong>Qualys VMDR</strong></td><td>Latest</td><td>Basic unauthenticated + Advanced SSH-credentialed scan</td></tr>
          <tr><td><strong>Greenbone OpenVAS</strong></td><td>Latest</td><td>Network vulnerability scan</td></tr>
          <tr><td><strong>OWASP ZAP</strong></td><td>v2.17.0</td><td>Active web application scan</td></tr>
          <tr><td><strong>Nikto</strong></td><td>v2.5.0</td><td>Web server scan</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <!-- METHODOLOGY -->
  <section>
    <h2>Methodology</h2>
    <p style="font-size:0.875rem;margin-bottom:1rem;">Aligned with <strong>NIST SP 800-115</strong> and the <strong>OWASP Testing Guide v4.2</strong>:</p>
    <ol class="steps">
      <li><strong>Reconnaissance &amp; Asset Identification</strong> &mdash; Port enumeration, OS and service fingerprinting.</li>
      <li><strong>Unauthenticated Scanning</strong> &mdash; Simulated external attacker perspective across both targets.</li>
      <li><strong>Credentialed Scanning (SSH)</strong> &mdash; Authenticated deep-dive on Metasploitable 2 via Nessus and Qualys.</li>
      <li><strong>Web Application Testing</strong> &mdash; Active scanning with ZAP, Nikto, and OpenVAS.</li>
      <li><strong>Finding Analysis &amp; Risk Scoring</strong> &mdash; CVSS v3.0 scoring, EPSS ratings, cross-tool deduplication and prioritisation.</li>
      <li><strong>Reporting</strong> &mdash; Consolidated findings for SOC analyst (technical) and senior leadership (executive summary + remediation roadmap).</li>
    </ol>
  </section>

  <!-- COMPLIANCE -->
  <section>
    <h2>Compliance Frameworks Referenced</h2>
    <div class="table-wrap">
      <table>
        <thead>
          <tr><th>Framework</th><th>Relevance</th></tr>
        </thead>
        <tbody>
          <tr><td><strong>PCI-DSS</strong></td><td>Multiple findings constitute likely violations around cardholder data and patch management.</td></tr>
          <tr><td><strong>GDPR</strong></td><td>Exposure of personal data for EU-resident business users.</td></tr>
          <tr><td><strong>SOX</strong></td><td>IT controls supporting financial reporting integrity.</td></tr>
          <tr><td><strong>NIST CSF</strong></td><td>Identify, Protect, Detect, Respond, Recover.</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <!-- REPO STRUCTURE -->
  <section>
    <h2>Repository Contents</h2>
    <pre>/
├── README.md
├── AFS-Presentation-Slides.pdf        ← Slide deck
├── AFS_SOC_VulnMgmt_Report.docx       ← Full assessment report
└── vuln-scans-reports/
    ├── Metasploitable 2/
    └── VulnWeb PHP App/</pre>
    <p style="font-size:0.8rem;color:var(--muted);">The <code>vuln-scans-reports/</code> folder contains raw scan outputs referenced as Appendix B in the main report.</p>
  </section>

  <!-- REMEDIATION -->
  <section>
    <h2>Remediation Summary</h2>
    <p style="font-size:0.875rem;">Top immediate actions <span class="badge badge-critical">&lt; 24 hours</span>:</p>
    <ol class="steps" style="margin-top:0.75rem;">
      <li><strong>Isolate <code>10.0.2.4</code></strong> from all networks immediately &mdash; treat as actively compromised.</li>
      <li><strong>Remove all four active backdoors</strong> &mdash; Ingreslock (port 1524), vsftpd 2.3.4 (port 6200), UnrealIRCd (ports 6667/6697), ProFTPD 1.3.3c (port 2121).</li>
      <li><strong>Upgrade PHP 5.6.40 to PHP 8.3.x</strong> on the VulnWeb application and delete <code>phpinfo.php</code>.</li>
      <li><strong>Replace all dynamic SQL</strong> with parameterised queries (PDO/MySQLi) across all web endpoints.</li>
    </ol>
  </section>

  <hr>

  <!-- DISCLAIMER -->
  <div class="disclaimer">
    <strong style="color:var(--text);">Disclaimer:</strong> This assessment was conducted in a controlled lab environment against intentionally vulnerable systems (Metasploitable 2) and a designated test web application (testphp.vulnweb.com). All findings should be validated through manual penetration testing prior to sign-off. Severity ratings use CVSS v3.0 where available; CVSS v2.0 is noted where used.
  </div>

</div>
</body>
</html>
