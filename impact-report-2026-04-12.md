# 💥 Impact Analysis Report

_Generated: 2026-04-12 11:59:54_

---

## Summary

| Target | Risk Level | Risk Score | Direct Callers | Dependent Packages |
|---|---|---|---|---|
| `index.html` | **LOW** | 15/100 | 0 | 0 |

## What is changing

The `index.html` file serves as the homepage and primary entry point for the SBG Doha MMA gym website (deployed at https://sbgdoha.netlify.app/). It presents the hero section with the gym's branding, main call-to-action ("Join Now" button linking to Classes.html), navigation to all other site sections, and a footer with links. Modifying or removing this file would break the website's primary landing page, home navigation links throughout the site, and the default entry point for visitors arriving at the root domain.

## Direct impact

**All Site Navigation (17 references across 6 files):**

- **About.html:22** - Logo link `<a href="index.html">` in navbar
- **About.html:25** - Home menu link in navigation list
- **About.html:90** - FAQ footer link
- **Classes.html:22** - Logo link in navbar
- **Classes.html:25** - Home menu link
- **Classes.html:104** - FAQ footer link
- **Contact.html:19** - Logo link in navbar
- **Contact.html:22** - Home menu link
- **Contact.html:66** - FAQ footer link
- **History.html:20** - Logo link in navbar
- **History.html:23** - Home menu link
- **History.html:154** - FAQ footer link
- **Trainers.html:21** - Logo link in navbar
- **Trainers.html:24** - Home menu link
- **Trainers.html:156** - FAQ footer link
- **index.html:26** - Self-referential logo link
- **index.html:29** - Self-referential Home menu link
- **index.html:64** - Self-referential FAQ footer link

Removing the file creates broken links site-wide. Changing the filename without updating all 17 references breaks navigation from every page back to the homepage.

## Indirect impact

**Web Server and Deployment Impact:**
- The Netlify deployment expects `index.html` as the default document at the root path `/`. Removing it would result in 404 errors for visitors accessing `https://sbgdoha.netlify.app/` directly
- Search engine indexed links pointing to the root domain would break
- Any external backlinks, social media shares, or bookmarks pointing to the homepage would fail
- The "Join Now" call-to-action flow (homepage → Classes.html) would be broken at the entry point

**Asset Dependencies:**
- `style.css` defines styles for `.Home`, `.banner`, `.highlights` classes used exclusively in index.html. While the stylesheet wouldn't break, these rules would become dead code
- `Script.js` (lines 2-5) adds sticky navbar behavior and navigation toggle that's used on index.html and all other pages
- Background image `./src/banner5.webp` referenced in style.css:95 is only displayed on the homepage

**User Journey Impact:**
The homepage is the primary conversion funnel entry point. Removing it breaks the intended user flow: discover gym → see hero message → click "Join Now" → view classes → contact/register.

## Critical paths

1. **External Traffic → index.html → Site Entry**
   - Users from search engines, social media, direct links arrive at root domain
   - Netlify serves index.html as default document
   - Breaking this blocks all new visitor acquisition
   - **Risk**: Complete loss of homepage functionality; primary business impact

2. **All Pages → index.html (Home/Logo Links) → Homepage**
   - Every page has 2-3 links back to index.html
   - Users expect logo/home clicks to return to start
   - Breaking this traps users on subpages with no return path
   - **Risk**: Violates fundamental web UX patterns; high user frustration

3. **index.html → Classes.html (Join Now CTA) → Conversion**
   - Hero section's primary call-to-action button
   - Main conversion funnel for gym membership signups
   - Breaking the homepage breaks the top-of-funnel
   - **Risk**: Direct revenue impact from lost lead generation

4. **Search Engines → index.html Metadata → SEO Ranking**
   - Title, meta tags, and content structure feed search indexing
   - Removing or significantly modifying impacts discoverability
   - **Risk**: Long-term organic traffic degradation

5. **index.html → style.css + Script.js → Visual/Interactive Presentation**
   - Changing class names or structure breaks CSS selectors
   - Modifying DOM structure breaks JavaScript event handlers
   - **Risk**: Silent functional regressions with no error messages

## Safety net

**Existing Protections:**
- Git version control provides rollback capability (currently on commit d885a47)
- Netlify deployment likely has rollback features and deploy previews
- The website is live and can be manually tested at https://sbgdoha.netlify.app/

**Critical Gaps:**
- **No automated tests**: Zero unit, integration, or end-to-end tests to catch regressions
- **No link validation**: No tooling to detect broken internal links across the 17 references
- **No CI/CD validation**: No pre-deploy checks for HTML validity, broken links, or asset references
- **No staging environment**: Changes appear to go directly to production (based on git history showing direct commits)
- **No monitoring**: No evidence of uptime monitoring, broken link detection, or analytics alerting
- **No type safety**: HTML/CSS/JS have no compile-time checking for reference integrity
- **No feature flags**: No gradual rollout mechanism for testing changes with subset of users
- **No visual regression testing**: No screenshot comparison to catch layout breaks

## Risk assessment

The pre-computed risk score of 15/100 (LOW) is a severe underestimate. The scoring appears designed for backend code analysis and completely misses the critical role of `index.html` as:
1. The production website's entry point
2. The target of 17 navigation references across all site pages  
3. The primary user acquisition and conversion funnel
4. The default document required by web server conventions

**Risk Upgrade Justification:**
- **Exported/Public**: The file is the public face of a live production website at a published URL
- **Critical System**: This is the primary customer-facing interface for a business
- **No Test Coverage**: Zero automated safety net; all regressions are manual-detect only
- **High Fan-In (Runtime)**: All pages link to it, all external traffic targets it; the metric incorrectly shows 0 because it's measuring code references, not hyperlinks
- **Silent Failures**: Broken links produce no compile-time errors
- **Immediate Production Impact**: Netlify deployment means changes go live immediately

The actual blast radius is SITE-WIDE with IMMEDIATE PRODUCTION IMPACT.

RISK: CRITICAL

## Testing strategy

**Before Merge:**
- Manually verify all 17 `href="index.html"` references across About.html, Classes.html, Contact.html, History.html, Trainers.html, and index.html still resolve correctly
- Test logo click navigation from every page returns to homepage
- Test "Home" menu item navigation from every page
- Test footer "FAQ" link behavior (currently points to index.html)
- Validate HTML using W3C validator or `html-validate` tool
- Check CSS class names (.Home, .banner, .highlights) still match style.css selectors
- Verify JavaScript event handlers still find DOM elements (navToggle onclick)
- Test responsive navbar behavior on mobile viewport
- Confirm external CDN resources (Font Awesome) still load
- Verify background image src/banner5.webp still loads

**Missing Tests to Add:**
- Automated link checker (e.g., `linkinator` or `broken-link-checker`) to validate all internal hrefs
- HTML validation in CI pipeline
- Visual regression tests using Percy, Chromatic, or BackstopJS for homepage hero section
- Basic Cypress or Playwright end-to-end test: visit root → verify hero content → click "Join Now" → verify Classes page loads
- Test that root path "/" serves index.html correctly

**Manual Validation Paths:**
- Load https://sbgdoha.netlify.app/ in multiple browsers (Chrome, Firefox, Safari, mobile)
- Navigate full user journey: homepage → Classes → back to home via logo
- Test mobile responsive navigation toggle functionality
- Verify hero section layout and "Join Now" button positioning
- Check footer links render correctly with icons

**Post-Deploy Smoke Checks:**
- Visit https://sbgdoha.netlify.app/ and verify page loads without 404
- Check browser console for JavaScript errors
- Verify no broken images or missing CSS
- Test navigation round-trip from 3 different pages back to home
- Confirm Google Search Console doesn't report new 404 errors within 24 hours
- Verify analytics (if configured) shows traffic reaching homepage

## Next steps

**If Modifying index.html:**
1. Create feature branch for changes
2. Make modifications to index.html
3. Run link validation across all 6 HTML files to confirm no broken hrefs
4. Test locally using `python -m http.server` or similar before deploying
5. Create Netlify deploy preview and manually test all navigation paths
6. If changing class names or IDs, update corresponding style.css selectors
7. If changing DOM structure, test Script.js functionality (navbar toggle, sticky behavior)
8. Deploy to production during low-traffic period
9. Monitor for 404 errors in Netlify logs for 24-48 hours
10. Have rollback plan ready (Netlify rollback or git revert to commit d885a47)

**If Renaming index.html:**
1. DO NOT rename without careful planning - this breaks web server conventions
2. If absolutely necessary, update all 17 references in About.html, Classes.html, Contact.html, History.html, Trainers.html, and index.html (self-references)
3. Configure Netlify redirect rule: `/ -> /new-name.html 301` in `_redirects` file
4. Update any external documentation, marketing materials, or social media links
5. Submit updated sitemap to Google Search Console

**If Removing index.html:**
1. DO NOT remove - this is the required entry point for the website
2. If decommissioning the site entirely, configure appropriate HTTP 410 (Gone) responses or redirect to new location
3. Update external references, backlinks, and search engine listings

**Monitoring After Deploy:**
- Check Netlify deploy logs for 404 errors
- Monitor Netlify analytics for traffic drop-offs (if available)
- Set up uptime monitoring with UptimeRobot or similar service
- Configure Google Search Console to alert on crawl errors
- Review user session recordings (if Hotjar/similar installed) for navigation issues

