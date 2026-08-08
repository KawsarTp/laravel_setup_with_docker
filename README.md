# Laravel Docker Development Environment

A Laravel application running completely inside Docker.

This project uses:

- Laravel
- PHP 8.4
- PHP-FPM
- Nginx
- MySQL 8.4
- Composer
- Docker Compose

The goal of this setup is to provide a consistent development environment for every developer in the team.

---

# Table of Contents

1. [Requirements](#1-requirements)
2. [Architecture](#2-architecture)
3. [Project Structure](#3-project-structure)
4. [Clone the Project](#4-clone-the-project)
5. [Environment Configuration](#5-environment-configuration)
6. [Docker Configuration](#6-docker-configuration)
7. [First Time Installation](#7-first-time-installation)
8. [Start the Application](#8-start-the-application)
9. [Access the Application](#9-access-the-application)
10. [Database Configuration](#10-database-configuration)
11. [Laravel Artisan Commands](#11-laravel-artisan-commands)
12. [Composer Commands](#12-composer-commands)
13. [Docker Commands](#13-docker-commands)
14. [View Logs](#14-view-logs)
15. [Access Containers](#15-access-containers)
16. [File Permissions](#16-file-permissions)
17. [Rebuilding Docker Images](#17-rebuilding-docker-images)
18. [Database Reset](#18-database-reset)
19. [Troubleshooting](#19-troubleshooting)
20. [Daily Development Workflow](#20-daily-development-workflow)
21. [Git Workflow](#21-git-workflow)
22. [New Developer Setup](#22-new-developer-setup)
23. [Important Docker Concepts](#23-important-docker-concepts)

---

# 1. Requirements

Only the following software is required on the host machine:

- Docker
- Docker Compose
- Git

You DO NOT need to install the following directly on your machine:

- PHP
- Composer
- MySQL
- Nginx

All of them are provided by Docker.

---

## Check Docker

Run:

```bash
docker --version