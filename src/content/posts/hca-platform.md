---
title: "Digital Platform for Happy Childhood Association"
published: 2025-06-22
description: "A Django-powered backend system supporting the Happy Childhood Association's digital platform, enabling streamlined communication, event tracking and members management."
image: ''
tags: ["Django","Python"]
category: 'Project'
draft: false 
lang: 'en'
---


**HCA Web** is the official digital platform for the **Happy Childhood Association**, a nonprofit based in El Atteuf, Ghardaïa, Algeria. The platform supports the association’s mission to care for children in need by providing a digital space for sharing projects,events and managing member interactions. I led the **backend development using Django**, building a secure, scalable REST API to power all site functionality.<br/>
**Link**: [hca-web](https://hca-web.vercel.app)


## Key Features

- **Secure Authentication** and role-based access with DRF and JWT  
- **Robust Admin Dashboard** for site management 


## Technologies Used

- **Python / Django** – Backend framework  
- **Django REST Framework (DRF)** – REST API structure  
- **JWT Authentication** – For secure user sessions  
- **PostgreSQL** – Relational DB for donations and project data hosted on [Render](https://render.com)  
- [**Cloudinary**](https://cloudinary.com) - for media storage
- **DRF Spectacular / Swagger** – Auto-generated, shareable API documentation  
- **Next.js (Front-End)** – By teammate, consuming REST API


## My Role & Contributions

- Built **secure, versioned APIs** for users, donations, and project management  
- Integrated **authentication and permissions**, including admin-only and writer-only routes  
- Created **API schema documentation** for seamless front-end integration  
- Ensured the backend was clean, maintainable, and production-ready


## Challenges & Learnings

- Implementing **multi-role access** (admin, donor, volunteer)  
- Coordinating live frontend-backend integration using real endpoints  
- Structuring models to support **future features**


## Outcome

- Full-stack app live at [hca-web](https://hca-web.vercel.app)  
- Backend designed to support future features like newsletter APIs, chat, or mobile integration  
