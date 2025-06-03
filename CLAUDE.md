# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the L402 Protocol documentation website built with Docusaurus. L402 is an HTTP-based payment protocol that enables machine-friendly payments using the HTTP 402 status code. The project contains:

- Protocol documentation and specifications
- Implementation examples and guides  
- A Docusaurus-based static site for hosting the documentation

The repository is currently migrating from an old site structure (`old_site/`) to a new Docusaurus-based architecture.

## Development Commands

### Package Manager
This project uses **npm** as the package manager.

### Essential Commands
```bash
# Install dependencies
npm

# Start development server with hot reload
npm start

# Build the static site
npm build

# Type checking
npm typecheck

# Serve built site locally
npm serve

# Deploy to GitHub Pages
npm deploy
```

### Development Server
- Development server runs on http://localhost:3000/
- Most changes are reflected live without restart
- Uses Docusaurus hot reload

## Architecture

### Content Structure
- `docs/` - Main documentation content (Markdown/MDX files)
- `blog/` - Blog posts and announcements  
- `src/` - React components and custom pages
- `static/` - Static assets (images, etc.)
- `old_site/` - Legacy site content being migrated

### Key Configuration Files
- `docusaurus.config.ts` - Main Docusaurus configuration
- `sidebars.ts` - Documentation sidebar structure (currently using auto-generated from filesystem)
- `package.json` - Contains all available npm scripts

### Site Configuration
- Uses Docusaurus v3.8.0 with TypeScript
- Auto-generates sidebar from docs folder structure
- Configured for GitHub Pages deployment
- Supports both docs and blog content types

## Development Notes

- Node.js 18.0+ required
- Site uses React 19 and modern TypeScript
- Configuration includes future flags for Docusaurus v4 compatibility
- Default configuration still contains placeholder content that may need customization for the L402 protocol