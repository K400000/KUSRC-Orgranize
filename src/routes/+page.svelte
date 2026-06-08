<script lang="ts">
  import organizations from '$lib/assets/organizations.json';

  type Organization = {
    id: number;
    type: string;
    nameTh: string;
    nameEn: string;
    descTh: string;
    descEn: string;
    banner: string;
    logo: string;
    contacts?: Record<string, string>;
  };

  const bannerMap = Object.fromEntries(
    Object.entries(import.meta.glob('$lib/assets/banners/*.{svg,png,jpg,jpeg}', { eager: true, import: 'default' }))
      .map(([path, asset]) => [path.split('/').pop() ?? path, String(asset)])
  ) as Record<string, string>;

  const logoMap = Object.fromEntries(
    Object.entries(import.meta.glob('$lib/assets/logos/*.{svg,png,jpg,jpeg}', { eager: true, import: 'default' }))
      .map(([path, asset]) => [path.split('/').pop() ?? path, String(asset)])
  ) as Record<string, string>;

  const filterLabels: Record<string, { th: string; en: string }> = {
    all: { th: 'ทั้งหมด', en: 'All' },
    'องค์กรนิสิต': { th: 'องค์กรนิสิต', en: 'Student Org' },
    สโมสร: { th: 'สโมสร', en: 'Council' },
    ชมรม: { th: 'ชมรม', en: 'Club' }
  };

  let currentLang = $state<'th' | 'en'>('th');
  let activeFilter = $state('all');
  let searchQuery = $state('');
  let selectedOrg = $state<Organization | null>(null);

  function setLang(lang: 'th' | 'en') {
    currentLang = lang;
  }

  function setFilter(type: string) {
    activeFilter = type;
  }

  function openDetails(org: Organization) {
    selectedOrg = org;
  }

  function closeDetails() {
    selectedOrg = null;
  }

  const filteredOrganizations = $derived((organizations as Organization[]).filter((org) => {
    const matchesType = activeFilter === 'all' || org.type === activeFilter;
    const haystack = `${org.nameTh} ${org.nameEn} ${org.type} ${org.descTh} ${org.descEn}`.toLowerCase();
    const matchesQuery = !searchQuery || haystack.includes(searchQuery.toLowerCase());
    return matchesType && matchesQuery;
  }));

  const resultLabel = $derived(filteredOrganizations.length > 0
    ? currentLang === 'th'
      ? `แสดง <b>${filteredOrganizations.length}</b> องค์กร`
      : `Showing <b>${filteredOrganizations.length}</b> organization${filteredOrganizations.length !== 1 ? 's' : ''}`
    : '');

  const modalContacts = $derived(
    selectedOrg
      ? (Object.entries(selectedOrg.contacts ?? {}) as [string, string][])
          .filter(([, value]) => value && value.trim().length > 0)
          .map(([label, url]) => ({
            label,
            url,
            icon: label.toLowerCase().includes('instagram') ? 'ig.png' : label.toLowerCase().includes('facebook') ? 'fb.svg' : null,
            isFacebook: label.toLowerCase().includes('facebook')
          }))
      : []
  );
</script>

<svelte:head>
  <title>องค์กรนิสิต — Kasetsart University Sriracha</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<nav>
  <a class="nav-brand" href="#top">
    <img class="nav-logo-img" src="https://admissions.src.ku.ac.th/assets/img/faviconx.png" alt="Kasetsart University Logo" onerror={() => {}} />
    <div class="nav-text-group">
      <b class="nav-university">Kasetsart University</b>
      <small class="nav-campus">Sriracha Campus</small>
    </div>
  </a>
  <div class="nav-sep"></div>
  <span class="nav-section-label">{currentLang === 'th' ? 'องค์กรนิสิต' : 'Student Organizations'}</span>
  <div class="nav-spacer"></div>
  <div class="lang-switch">
    <button class:active={currentLang === 'th'} class="lang-btn" onclick={() => setLang('th')}>TH</button>
    <button class:active={currentLang === 'en'} class="lang-btn" onclick={() => setLang('en')}>EN</button>
  </div>
</nav>

<section class="hero" id="top">
  <div class="hero-eyebrow">Kasetsart University · Sriracha Campus</div>
  <h1>{currentLang === 'th' ? 'ค้นหาองค์กร' : 'Find Organizations'}</h1>
  <p class="hero-sub">{currentLang === 'th'
    ? 'ศูนย์รวมองค์กร สโมสร และชมรมนิสิต มหาวิทยาลัยเกษตรศาสตร์ วิทยาเขตศรีราชา'
    : 'Directory of student organizations, clubs, and associations at Kasetsart University Sriracha Campus'}</p>
  <div class="search-container">
    <div class="search-icon-wrap">
      <svg viewBox="0 0 24 24" fill="none" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <circle cx="11" cy="11" r="8"></circle>
        <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
      </svg>
    </div>
    <input class="search-box" type="search" placeholder={currentLang === 'th' ? 'ค้นหาชื่อองค์กร หรือประเภท...' : 'Search organizations or type...'} bind:value={searchQuery} />
  </div>
</section>

<div class="content">
  <div class="toolbar">
    <div class="filters">
      {#each Object.keys(filterLabels) as key}
        <button class:active={activeFilter === key} class="filter-btn" onclick={() => setFilter(key)}>{currentLang === 'th' ? filterLabels[key].th : filterLabels[key].en}</button>
      {/each}
    </div>
    <span class="result-count">{@html resultLabel}</span>
  </div>

  <div class="grid">
    {#each filteredOrganizations as org}
      <article class="card">
        <div class="card-banner" class:no-banner={!org.banner || !bannerMap[org.banner]}>
          {#if org.banner && bannerMap[org.banner]}
            <img class="banner-img" src={bannerMap[org.banner]} alt="" onerror={(event) => { (event.currentTarget as HTMLImageElement).style.display = 'none'; }} />
          {/if}
          <div class="banner-scrim"></div>
          <div class="card-logo"><img src={logoMap[org.logo]} alt={org.nameTh} /></div>
          <span class:badge-org={org.type === 'องค์กรนิสิต'} class:badge-assoc={org.type === 'สโมสร'} class:badge-club={org.type === 'ชมรม'} class="type-badge">{org.type}</span>
        </div>
        <div class="card-body">
          <div class="card-name">{currentLang === 'th' ? org.nameTh : org.nameEn}</div>
          <div class="card-desc">{currentLang === 'th' ? org.descTh : org.descEn}</div>
        </div>
        <div class="card-footer">
          <button class="btn-detail" onclick={() => openDetails(org)}>
            {currentLang === 'th' ? 'ดูรายละเอียด' : 'View Details'}
            <svg viewBox="0 0 24 24" fill="none" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"></line><polyline points="12 5 19 12 12 19"></polyline></svg>
          </button>
        </div>
      </article>
    {/each}

    {#if filteredOrganizations.length === 0}
      <div class="empty show">
        <div class="empty-icon">🌿</div>
        <div class="empty-text">{currentLang === 'th' ? 'ไม่พบองค์กรที่ตรงกับคำค้นหา' : 'No organizations found.'}</div>
      </div>
    {/if}
  </div>
</div>

{#if selectedOrg}
  <div
    class="modal-backdrop"
    role="button"
    tabindex="0"
    onclick={() => closeDetails()}
    onkeydown={(event) => {
      if (event.key === 'Escape' || event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        closeDetails();
      }
    }}
  >
    <article class="modal-card" role="dialog" aria-modal="true" aria-label="Organization details" onclick={(event) => event.stopPropagation()}>
      <button class="modal-close" type="button" aria-label="Close details" onclick={() => closeDetails()}>×</button>
      <div class="modal-logo-wrap">
        <img class="modal-logo" src={logoMap[selectedOrg.logo]} alt={selectedOrg.nameTh} />
      </div>
      <h3 class="modal-title">{currentLang === 'th' ? selectedOrg.nameTh : selectedOrg.nameEn}</h3>

      {#if modalContacts.length > 0}
        <div class="modal-contact-box">
          <h4>{currentLang === 'th' ? 'ติดต่อ' : 'Contact'}</h4>
          <div class="modal-links">
            {#each modalContacts as item}
              <a class="contact-link" href={item.url} target="_blank" rel="noreferrer">
                {#if item.icon}
                  <img class="contact-icon" class:facebook-icon={item.isFacebook} src={logoMap[item.icon]} alt="" />
                {/if}
                <span>{item.label}</span>
              </a>
            {/each}
          </div>
        </div>
      {/if}
    </article>
  </div>
{/if}

<footer>
  <p>
    <span>{currentLang === 'th' ? 'สร้างโดย' : 'Created by'}</span>
    <a href="https://www.instagram.com/cybergeekclub.src/">Cyber Geek Club</a>
    <span>{currentLang === 'th' ? 'เพื่อ Kasetsart University Sriracha Campus' : 'for Kasetsart University Sriracha Campus'}</span>
  </p>
  <p style="margin-top:.3rem">© 2024 Kasetsart University Sriracha Campus. All rights reserved.</p>
</footer>

<style>
  :global(body) { margin: 0; font-family: 'Sarabun', Arial, sans-serif; background: var(--ku-paler, #f4fbf6); color: #0f2d1c; }
  :global(*) { box-sizing: border-box; }
  nav { position: sticky; top: 0; z-index: 200; min-height: 64px; background: rgba(10,34,24,.97); backdrop-filter: blur(20px) saturate(1.5); border-bottom: 1px solid rgba(200,168,75,.15); display: flex; align-items: center; padding: .75rem clamp(1rem, 5vw, 2.8rem); gap: .9rem; }
  .nav-brand { display: flex; align-items: center; gap: .75rem; text-decoration: none; flex-shrink: 0; min-width: 0; }
  .nav-logo-img { width: 40px; height: 40px; border-radius: 50%; object-fit: contain; background: rgba(255,255,255,.06); padding: 2px; flex-shrink: 0; }
  .nav-text-group { line-height: 1.15; }
  .nav-university { display: block; color: #fff; font-size: .82rem; font-weight: 700; letter-spacing: .01em; white-space: nowrap; }
  .nav-campus { display: block; color: rgba(255,255,255,.4); font-size: .63rem; font-weight: 500; letter-spacing: .1em; text-transform: uppercase; white-space: nowrap; }
  .nav-sep { width: 1px; height: 28px; background: rgba(255,255,255,.12); flex-shrink: 0; }
  .nav-section-label { font-size: .72rem; font-weight: 600; letter-spacing: .1em; text-transform: uppercase; color: #c8a84b; white-space: nowrap; flex-shrink: 0; }
  .nav-spacer { flex: 1 1 auto; min-width: 8px; }
  .lang-switch { display: flex; align-items: center; background: rgba(255,255,255,.07); border: 1px solid rgba(255,255,255,.1); border-radius: 999px; overflow: hidden; flex-shrink: 0; margin-left: auto; }
  .lang-btn { padding: .35rem .75rem; font-family: 'Sarabun', sans-serif; font-size: .75rem; font-weight: 700; letter-spacing: .04em; color: rgba(255,255,255,.45); background: transparent; border: none; cursor: pointer; transition: all .18s; line-height: 1; }
  .lang-btn.active { background: #c8a84b; color: #0a2218; border-radius: 999px; }
  .lang-btn:not(.active):hover { color: rgba(255,255,255,.8); }

  .hero { background: linear-gradient(160deg, #0a2218 0%, #1d5235 60%, #286644 100%); padding: clamp(3.5rem,9vw,5.5rem) clamp(1rem,5vw,2.5rem) clamp(2.5rem,7vw,4rem); display: flex; flex-direction: column; align-items: center; text-align: center; position: relative; overflow: hidden; }
  .hero::before { content: ''; position: absolute; inset: 0; background-image: repeating-linear-gradient(0deg, transparent, transparent 39px, rgba(255,255,255,.022) 40px), repeating-linear-gradient(90deg, transparent, transparent 39px, rgba(255,255,255,.022) 40px); pointer-events: none; }
  .hero-eyebrow { position: relative; font-size: .68rem; font-weight: 700; letter-spacing: .14em; text-transform: uppercase; color: #c8a84b; margin-bottom: 1rem; }
  .hero h1 { position: relative; font-size: clamp(1.9rem, 5vw, 3rem); font-weight: 800; color: #fff; line-height: 1.15; letter-spacing: -.01em; margin-bottom: .9rem; }
  .hero-sub { position: relative; font-size: clamp(.88rem, 2vw, 1rem); color: rgba(255,255,255,.5); max-width: 420px; line-height: 1.75; margin-bottom: 2.4rem; }
  .search-container { position: relative; width: 100%; max-width: 480px; }
  .search-box { width: 100%; height: 52px; background: rgba(255,255,255,.1); border: 1.5px solid rgba(255,255,255,.18); border-radius: 999px; padding: 0 1.25rem 0 3.2rem; color: #fff; font-family: 'Sarabun', sans-serif; font-size: .95rem; outline: none; transition: background .2s, border-color .2s, box-shadow .2s; -webkit-appearance: none; appearance: none; }
  .search-box::placeholder { color: rgba(255,255,255,.35); }
  .search-box:focus { background: rgba(255,255,255,.15); border-color: rgba(200,168,75,.7); box-shadow: 0 0 0 3px rgba(200,168,75,.14); }
  .search-icon-wrap { position: absolute; left: 1.05rem; top: 50%; transform: translateY(-50%); pointer-events: none; display: flex; align-items: center; }
  .search-icon-wrap svg { width: 18px; height: 18px; stroke: rgba(255,255,255,.4); }

  .content { max-width: 1100px; margin: 0 auto; padding: 0 clamp(1rem, 4vw, 2rem); }
  .toolbar { display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: .75rem; padding: 1.8rem 0 1.2rem; }
  .filters { display: flex; gap: .5rem; flex-wrap: wrap; }
  .filter-btn { height: 36px; padding: 0 1.1rem; border-radius: 999px; border: 1.5px solid #b0d0be; background: #fff; color: #5a8a6a; font-family: 'Sarabun', sans-serif; font-size: .82rem; font-weight: 600; cursor: pointer; transition: all .18s; white-space: nowrap; }
  .filter-btn:hover { border-color: #3a8a5e; color: #1d5235; background: #e8f5ee; }
  .filter-btn.active { background: #1d5235; border-color: #1d5235; color: #fff; box-shadow: 0 2px 8px rgba(29,82,53,.25); }
  .result-count { font-size: .82rem; color: #8aad98; font-weight: 500; white-space: nowrap; }

  .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(min(100%, 300px), 1fr)); gap: clamp(1rem, 2.5vw, 1.5rem); padding-bottom: clamp(3rem, 8vw, 5rem); }
  .card { background: #fff; border-radius: 16px; overflow: hidden; border: 1px solid #d0e8d8; box-shadow: 0 1px 3px rgba(10,34,24,.06), 0 4px 16px rgba(10,34,24,.08); display: flex; flex-direction: column; transition: transform .28s cubic-bezier(.34,1.4,.64,1), box-shadow .28s; border-radius: 16px; }
  .card:hover { transform: translateY(-5px); box-shadow: 0 8px 32px rgba(10,34,24,.16), 0 2px 8px rgba(10,34,24,.08); }
  .card-banner { position: relative; height: 150px; overflow: hidden; background: #133526; }
  .card-banner.no-banner { background: #ffffff; }
  .card-banner img.banner-img { width: 100%; height: 100%; object-fit: cover; display: block; transition: transform .5s ease; }
  .card:hover .banner-img { transform: scale(1.05); }
  .banner-scrim { position: absolute; inset: 0; background: linear-gradient(to bottom, rgba(10,34,24,0) 50%, rgba(10,34,24,.55) 100%); }
  .card-logo { position: absolute; top: .75rem; left: .75rem; width: 56px; height: 56px; border-radius: 14px; border: 2.5px solid rgba(255,255,255,.92); overflow: hidden; background: rgba(19,53,38,.96); box-shadow: 0 10px 18px rgba(0,0,0,.18); z-index: 2; }
  .card-logo img { width: 100%; height: 100%; object-fit: contain; }
  .type-badge { position: absolute; top: .7rem; right: .7rem; padding: .28rem .7rem; border-radius: 999px; font-size: .68rem; font-weight: 700; letter-spacing: .05em; text-transform: uppercase; backdrop-filter: blur(8px); }
  .badge-org { background: rgba(10,34,24,.72); color: #9fe8be; border: 1px solid rgba(159,232,190,.25); }
  .badge-club { background: rgba(180,130,0,.75); color: #ffeea0; border: 1px solid rgba(255,238,160,.25); }
  .badge-assoc { background: rgba(20,60,130,.72); color: #b8d4ff; border: 1px solid rgba(184,212,255,.25); }
  .card-body { padding: 1.05rem 1.1rem 0; flex: 1; display: flex; flex-direction: column; gap: .45rem; }
  .card-name { font-size: .97rem; font-weight: 700; color: #0f2d1c; line-height: 1.35; }
  .card-desc { font-size: .81rem; color: #5a8a6a; line-height: 1.65; display: -webkit-box; -webkit-line-clamp: 2; line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; flex: 1; }
  .card-footer { padding: 1rem 1.2rem 1.2rem; }
  .btn-detail { display: flex; align-items: center; justify-content: center; gap: .45rem; width: 100%; height: 40px; border-radius: 8px; background: #f4fbf6; border: 1.5px solid #b0d0be; color: #1d5235; font-family: 'Sarabun', sans-serif; font-size: .84rem; font-weight: 700; cursor: pointer; transition: all .2s; }
  .btn-detail:hover { background: #1d5235; border-color: #1d5235; color: #fff; }
  .btn-detail svg { width: 14px; height: 14px; stroke: currentColor; flex-shrink: 0; transition: transform .2s; }
  .btn-detail:hover svg { transform: translateX(3px); }
  .empty { display: none; flex-direction: column; align-items: center; justify-content: center; gap: .75rem; padding: 5rem 2rem; text-align: center; grid-column: 1 / -1; }
  .empty.show { display: flex; }
  .empty-icon { font-size: 2.5rem; opacity: .35; }
  .empty-text { color: #8aad98; font-size: .92rem; }

  .modal-backdrop { position: fixed; inset: 0; background: rgba(7, 18, 12, .72); display: flex; align-items: center; justify-content: center; padding: 1rem; z-index: 500; }
  .modal-card { width: min(100%, 420px); max-height: calc(100vh - 2rem); overflow: auto; background: linear-gradient(180deg, #ffffff 0%, #eef7f0 100%); border: 1px solid #d8ebe0; border-radius: 24px; box-shadow: 0 24px 60px rgba(0,0,0,.25); padding: 1rem; position: relative; }
  .modal-close { position: absolute; top: .75rem; right: .75rem; width: 36px; height: 36px; border-radius: 999px; border: none; background: rgba(15,45,28,.08); color: #10331d; font-size: 1.1rem; cursor: pointer; }
  .modal-logo-wrap { display: flex; justify-content: center; margin-top: .2rem; margin-bottom: .85rem; }
  .modal-logo { width: 88px; height: 88px; object-fit: contain; border-radius: 18px; background: #fff; border: 1px solid #dbeee0; padding: .35rem; box-shadow: 0 8px 18px rgba(10,34,24,.08); }
  .modal-title { margin: 0 0 .4rem; font-size: 1.05rem; color: #0f2d1c; text-align: center; }
  .modal-desc { margin: 0; font-size: .92rem; line-height: 1.65; color: #4f6f5d; text-align: center; }
  .modal-contact-box { margin-top: 1rem; padding-top: .9rem; border-top: 1px solid #dbeee0; }
  .modal-contact-box h4 { margin: 0 0 .55rem; font-size: .82rem; color: #1d5235; text-transform: uppercase; letter-spacing: .12em; }
  .modal-links { display: flex; flex-direction: column; gap: .55rem; }
  .contact-link { display: flex; align-items: center; gap: .75rem; width: 100%; min-height: 54px; padding: .75rem .9rem; border-radius: 16px; background: linear-gradient(135deg, #1d5235 0%, #2d6a47 100%); color: #fff; text-decoration: none; font-size: .95rem; font-weight: 700; box-shadow: 0 10px 18px rgba(29,82,53,.18); }
  .contact-icon { width: 32px; height: 32px; object-fit: contain; flex-shrink: 0; }
  .contact-icon.facebook-icon { filter: brightness(0) saturate(100%) invert(27%) sepia(98%) saturate(1478%) hue-rotate(179deg) brightness(96%) contrast(101%); }

  @media (max-width: 900px) {
    .toolbar { align-items: flex-start; }
    .result-count { width: 100%; }
  }

  @media (max-width: 760px) {
    nav { min-height: 82px; padding: .65rem .8rem; flex-wrap: nowrap; align-items: center; gap: .35rem; }
    .nav-brand {
      order: 2;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      justify-content: center;
      min-width: 0;
      z-index: 1;
    }
    .nav-logo-img { width: 52px; height: 52px; }
    .nav-text-group { display: none; }
    .nav-sep, .nav-section-label { display: none; }
    .nav-spacer { display: none; }
    .lang-switch { order: 3; margin-left: auto; align-self: center; padding: .08rem; position: relative; z-index: 2; }
    .lang-btn { padding: .35rem .55rem; font-size: .72rem; min-width: 2.4rem; }
    .hero { padding-top: 3rem; }
    .hero h1 { font-size: clamp(1.7rem, 8vw, 2.25rem); }
    .hero-sub { margin-bottom: 1.6rem; }
    .toolbar { padding-top: 1.2rem; }
    .filters { width: 100%; overflow-x: auto; padding-bottom: .2rem; }
    .filter-btn { flex: 0 0 auto; }
    .grid { grid-template-columns: 1fr; }
    .card-banner { height: 132px; }
    .card-logo { width: 54px; height: 54px; top: .65rem; left: .65rem; }
    .card-body { padding-top: .95rem; }
    .card-footer { padding-top: .9rem; }
  }

  @media (max-width: 520px) {
    nav { padding: .6rem .7rem; min-height: 78px; }
    .nav-brand { position: absolute; left: 50%; transform: translateX(-50%); }
    .nav-logo-img { width: 56px; height: 56px; }
    .lang-switch { flex: 0 0 auto; margin-left: auto; position: relative; z-index: 2; }
    .lang-btn { padding: .32rem .48rem; font-size: .7rem; min-width: 2.1rem; }
    .hero { padding-inline: .9rem; }
    .content { padding: 0 .9rem; }
    .filter-btn { height: 34px; font-size: .78rem; padding: 0 .9rem; }
    .card-name { font-size: .95rem; }
    .card-desc { font-size: .8rem; }
  }

  footer { background: #0a2218; border-top: 1px solid rgba(200,168,75,.15); padding: 1.6rem clamp(1rem,5vw,2.5rem); text-align: center; }
  footer p { font-size: .78rem; color: rgba(255,255,255,.35); line-height: 1.7; }
  footer a { color: #c8a84b; text-decoration: none; font-weight: 600; }
  footer a:hover { color: #e8cc7a; }
</style>
