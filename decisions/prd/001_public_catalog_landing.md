# PRD - Public Catalog Landing v1

## Document Control

- Status: Draft
- Version: 0.1
- Date: 2026-03-13
- Owner: Product

## Executive Summary

The goal of this first iteration is to give the publisher a credible online presence through a public website where visitors can discover the catalog and consult relevant information about books, collections, and authors without authentication.

This iteration is not intended to solve ecommerce, editorial CMS, advanced marketing automation, or backoffice workflows. Its purpose is to make the catalog visible, navigable, and useful for visitors who want to understand what the publisher offers and access the information they are looking for with low friction.

## Background and Context

The current backend is designed primarily as a secured catalog management system for internal use. It exposes CRUD-oriented endpoints for books, authors, and collections, protected with JWT and roles.

In the coming weeks, a public web client will be implemented as the editorial landing page. That frontend needs a public, read-only consumption model, aligned with public visibility rules and independent of backoffice concerns.

This PRD defines the product scope and user needs for that public experience so that later technical decisions, including API contract design, can be derived from explicit product intent.

## Problem Statement

Today, the editorial does not yet have a public digital surface where a visitor can:

- Understand the editorial identity at a glance.
- Browse the available catalog naturally.
- Access complete and reliable information about a book.
- Discover the relationship between books, authors, and collections.
- Reach relevant content without needing credentials or internal knowledge.

Without this public layer, the catalog exists operationally but not effectively as a discovery product.

## Product Vision

Create a first public web experience that presents the editorial and its catalog in a simple, trustworthy, and usable way, so that any visitor can browse published titles, explore related entities, and find the information they expect from a professional publisher website.

## Goals

- Provide a public and frictionless catalog browsing experience.
- Make published books easy to discover and consult.
- Give each published book a complete detail page with its relevant metadata.
- Strengthen the editorial's online presence through a professional landing page.
- Reuse the existing catalog domain as the source of truth while exposing a public-facing view of the data.

## Non-Goals

- Ecommerce, checkout, cart, or payment flows.
- User accounts, favorites, reviews, or personalization.
- Editorial CMS capabilities for marketing pages.
- Advanced catalog search or recommendation engines.
- Content workflows more complex than visibility of already published catalog data.

## Target Users

### Primary User

Visitor interested in the publisher's catalog who wants to discover books or consult information about a specific title.

### Secondary Users

- Readers arriving from social, search, events, or direct links.
- Teachers, families, librarians, or professionals evaluating the catalog.
- People who want to learn about an author or collection before deciding what to read or recommend.

## User Needs and Jobs To Be Done

When a visitor accesses the website, they need to:

- Understand quickly what the publisher offers.
- Browse the catalog without logging in.
- Find books by navigation and contextual links.
- Open a book page and see enough information to evaluate it.
- Jump from a book to its author or collection and back.
- Trust that the information shown is current and consistent.

Core jobs to be done:

- "Help me discover what books this publisher has."
- "Help me confirm details about a specific book."
- "Help me understand who wrote the book and what collection it belongs to."
- "Help me navigate the catalog naturally without prior knowledge."

## Product Principles

- Public first: no authentication for catalog consultation.
- Clarity over density: visitors should not face backoffice-oriented data or jargon.
- Navigation over complexity: simple browsing should solve most needs in v1.
- Trustworthy content: only data safe and intentional for public exposure should appear.
- Reuse with discipline: leverage the existing catalog core, but do not leak internal contracts directly.

## Scope for v1

### In Scope

- Public landing page introducing the publisher and linking into the catalog.
- Public listing pages for books, authors, and collections.
- Public detail pages for books, authors, and collections.
- Cross-navigation between related entities.
- Public visibility rules for what can and cannot be shown.
- Responsive experience for desktop and mobile.
- SEO-friendly public pages and metadata.

### Out of Scope

- Purchases or direct sales.
- Editorial blog or news system.
- Search autocomplete, faceted search, or recommendation rails.
- Multi-language public experience.
- User-generated content.

## Functional Requirements

### FR-1 Public Landing Page

The website must include a landing page that:

- Presents the editorial brand and purpose.
- Offers clear entry points into the catalog.
- Highlights a small curated or automatic sample of catalog content.
- Works as the main access point for first-time visitors.

### FR-2 Public Book Listing

Visitors must be able to open a public listing of books that:

- Shows only publicly visible books.
- Supports pagination.
- Displays enough summary information to decide whether to open a detail page.
- Links clearly to the book detail.

Minimum summary information per book:

- Title
- Cover image
- Author name
- Collection name
- Short descriptive snippet if available
- Public price if the business decides to expose it

### FR-3 Public Book Detail

Each visible book must have a public detail page with the information a visitor reasonably expects when consulting a publisher catalog.

Minimum detail information:

- Title
- Cover image
- Author
- Collection
- Description
- ISBN
- Publication date
- Page count
- Language information when relevant
- Genre information when relevant
- Public price if approved for exposure

The book detail page must also link to:

- The corresponding author page
- The corresponding collection page

### FR-4 Public Author Listing and Detail

Visitors must be able to browse authors and consult an author detail page.

Minimum public author information:

- Full name
- Pseudonym if relevant
- Biography if available
- Website if available
- Related published books

### FR-5 Public Collection Listing and Detail

Visitors must be able to browse collections and consult a collection detail page.

Minimum public collection information:

- Collection name
- Description if available
- Related published books

### FR-6 Contextual Navigation

The public experience must support natural navigation across entities:

- From landing page to catalog sections
- From book to author
- From book to collection
- From author to related books
- From collection to related books

The user should be able to reach a desired published book in a small number of interactions without relying on internal identifiers or private URLs.

### FR-7 Public Visibility Rules

The public catalog must not expose all internal catalog records by default.

Initial visibility rules for v1:

- Only books with public publication status should be visible.
- Draft or otherwise non-public books must never appear in public listings or detail pages.
- Authors and collections should only be public if they are associated with at least one visible book, unless a future explicit visibility rule is introduced.

### FR-8 Stable Public URLs

Public pages must have stable and human-friendly URLs suitable for sharing and indexing.

This implies that public routing should not rely exclusively on internal UUIDs if friendlier public identifiers such as slugs are introduced.

### FR-9 Empty and Incomplete States

The experience must degrade gracefully when:

- There are no visible books yet.
- An author or collection has little metadata.
- A requested public resource does not exist or is not public.

The visitor must receive a clear, non-technical response.

## Content and Data Requirements

The first iteration depends on having enough quality public data to avoid a catalog that feels broken or incomplete.

Required data quality expectations:

- Each visible book should have at least title, author, collection, and a usable cover image or safe fallback.
- Descriptions should be present for most published books.
- Author biography and collection description are optional for launch but strongly recommended.
- Public-facing text must be curated enough to represent the editorial professionally.

## Information Architecture

Proposed top-level public structure:

- Home
- Books
- Authors
- Collections
- Book detail
- Author detail
- Collection detail

Expected core navigation model:

- Home → Books / Authors / Collections
- Books → Book detail
- Book detail → Author detail / Collection detail
- Author detail → Book detail
- Collection detail → Book detail

## UX and Quality Requirements

- The public website must be responsive and usable on mobile and desktop.
- Catalog navigation must be understandable without prior context.
- The experience should privilege readability, visual trust, and discoverability.
- Page performance must feel fast enough for casual browsing.
- Public pages should be indexable and shareable.

## SEO Requirements

The first iteration should support basic discoverability from search engines and link sharing.

Minimum SEO expectations:

- Unique page titles for public detail pages
- Meaningful meta descriptions when possible
- Stable canonical public URLs
- Pages accessible without authentication

## Success Metrics

This iteration will be considered successful if:

- The publisher has a public web presence with accessible catalog pages.
- A visitor can browse the visible catalog without authentication.
- A visitor can reach any public book detail page and understand the book from the information shown.
- No draft or non-public catalog data is exposed publicly.
- The main public navigation works reliably on desktop and mobile.

## Technical Implications for Backend and API

This PRD is product-focused, but it implies the following backend needs:

- A separate public, read-only API surface.
- Anonymous access for public catalog consumption.
- Public response models decoupled from backoffice response models.
- Explicit visibility rules, especially around publication status.
- Pagination support for public listings.
- Stable public identifiers if the frontend uses slugs.
- Safe handling of not found and not public resources.
- Cache-friendly GET endpoints where possible.

## Risks and Dependencies

- Catalog quality risk: the experience will feel weak if descriptions, covers, or author/collection metadata are missing.
- Visibility risk: exposing internal-only records would damage trust and product clarity.
- SEO risk: if URLs and metadata are weak, discoverability will be limited.
- Product ambiguity risk: unresolved questions about price exposure, search, and featured content can delay contract definition.

## Open Questions

- Do we need basic search in v1, or is browse-first navigation enough for launch?
- Will featured books and collections be curated manually or derived automatically?
- Should public routes use slugs from day one?
- What is the minimum metadata required for a book to be considered launch-ready?

## Launch Readiness Checklist

- Public visibility rules are implemented and tested.
- Enough catalog content exists to avoid empty-feeling pages.
- Public landing page links correctly into all public catalog sections.
- Public detail pages show only approved fields.
- Anonymous access works for all intended public routes.
- Error and empty states are clear and non-technical.

## Recommendation for the Next Step

Before defining the public API contract, the team should validate the open questions above and freeze a v1 product scope for:

- Public entities to expose
- Public fields per entity
- Visibility rules
- Public URL strategy
- Whether search and public price belong to the MVP
