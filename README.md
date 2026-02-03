# TribeCRM Agent Documentation

This repository contains documentation and workflow guides for the Claude Code agent working with TribeCRM projects.

## TribeCRM Platform Overview

The TribeCRM platform consists of 5 projects:

1. **tribecrm-api** - Backend (Java/Spring Boot) with dynamic entity system
2. **tribecrm-app** - Frontend (React/TypeScript) with Material-UI
3. **tribecrm-authentication** - OAuth/SSO service (Node.js/Express with ORY Hydra)
4. **tribecrm-test-automation-qa** - E2E and API tests (Cypress)
5. **tribecrm-agent** - Documentation and AI workflows (this repository)

## Contents

### [Tribe_platform_tech_description.md](Tribe_platform_tech_description.md)
**Tribe Platform Technical Description**

Comprehensive technical documentation covering:
- **Project Overview**: 5 projects (API, App, Auth, Tests, Agent) with technology stack details
- **Architecture**: Dynamic entity system, multi-tenant CRM, OData 4.10, OAuth2/SSO
- **Development Workflows**: Git conventions, local setup, building, testing
- **Deployment**: GKE and App Engine deployment procedures
- **Troubleshooting**: Common issues and solutions for all projects

This document provides complete technical context about the TribeCRM platform structure, architecture, conventions, and operational procedures.

### [Tribe_Autonomous_Ticket_delivery.md](Tribe_Autonomous_Ticket_delivery.md)
**Tribe Autonomous Ticket Delivery Workflow**

Standard workflow documentation for autonomous AI-assisted development:
1. **Create Jira Ticket** - Required fields, metadata, AI-generated code notice
2. **Create Branch** - Branch naming conventions
3. **Implement Changes** - Code review checklist
4. **Test with Playwright** - Mandatory testing before commits
5. **Commit Changes** - Conventional Commits format
6. **Push to Remote** - Git push procedures
7. **Create Pull Request** - Base branch verification (master-rsbuild)
8. **Update Jira Ticket** - Status transitions and field verification

Includes checklists, examples, and important notes on Git safety, Jira requirements, GitHub requirements, and testing requirements.

## Purpose

This repository serves as the central knowledge base for Claude Code when working on TribeCRM projects. It ensures:
- Consistent development practices across all TribeCRM projects
- Proper integration with Jira, GitHub, and testing workflows
- Compliance with team conventions and quality standards
- Transparency about AI-generated code contributions

## Usage

These documents are referenced by Claude Code during development sessions to:
- Understand the TribeCRM workspace structure
- Follow standardized workflows for tickets, branches, commits, and PRs
- Apply proper testing procedures with Playwright MCP
- Maintain code quality and team conventions

## Maintenance

This documentation should be updated when:
- Workflow processes change
- New projects are added to the workspace
- Team conventions evolve
- Tool configurations are modified

---

**Organization**: efficy-sa
**Team**: Tribe
**Last Updated**: 2026-02-03
