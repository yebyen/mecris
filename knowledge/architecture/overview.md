---
type: Architecture
title: Mecris System Architecture Overview
description: Mecris is a personal accountability system with Python MCP server, Go services, Android client, and Spin cloud deployment. Core loop: Orient → Plan → Archive → PR-Test.
generated: { by: agent/cli, at: 2026-09-06T15:36:07Z }
---

# Related Concepts
- [MCP Server (mcp_server.py)](mcp-server.md): MCP Server is the primary integration layer for all system components
- [Go Services (mecris-go, mecris-go-spin, mecris-go-project)](go-services.md): Go services provide performance-critical backend functionality
- [Beeminder Goal Integration](beeminder-integration.md): Beeminder integration provides goal tracking and accountability mechanisms
- [Urbit Gall Agent Pattern Loop](gall-loop.md): Gall loop orchestrates the core agent development cycle
- [Specialized Technical Skills Integration](specialized-skills.md): Specialized skills extend agent capabilities for specific domains
- [Prometheus Status Skill](infrastructure/prometheus-status.md): Prometheus Status skill provides monitoring infrastructure observability
- [Alertmanager Install Skill](infrastructure/alertmanager-install.md): Alertmanager Install skill manages alerting infrastructure for system notifications
- [Flux Status Skill](infrastructure/flux-status.md): Flux Status skill provides GitOps deployment observability
- [Kubeconfig Setup Skill](infrastructure/kubeconfig-setup.md): Kubeconfig Setup skill manages Kubernetes access for service deployment
- [Mecris Architectural Philosophy: The Diseased Forest](philosophy.md): Architectural philosophy defines Mecris as a parasite feeding on human failure with cloud/local duality
- [Neon DB - The Forest Floor](data/neon-db.md): Neon DB serves as the persistent storage layer (Forest Floor) for all Mecris data
- [Iron Heart - Akamai/Fermyon Cloud Machines](infrastructure/iron-heart.md): Iron Heart provides scalable cloud backend capacity for Mecris services
- [Standard of Bone - JSON/WIT Communication Language](standards/standard-of-bone.md): Standard of Bone enables JSON/WIT communication between all Mecris components
