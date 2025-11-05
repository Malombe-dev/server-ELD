# ELD Log Generator - Backend Server

A Django REST API backend for Electronic Logging Device (ELD) compliance and trip planning system.

## Features

- 🚛 HOS (Hours of Service) compliant route calculation
- 📍 Automated stop planning with fuel and rest breaks
- 📄 ELD log generation and management
- 🗺️ Integration with OpenRouteService for routing
- 📊 PDF report generation
- 🔄 RESTful API endpoints

## Tech Stack

- **Framework**: Django 4.2+ & Django REST Framework
- **Database**: SQLite (development) / PostgreSQL (production)
- **APIs**: OpenRouteService, Nominatim Geocoding
- **PDF Generation**: ReportLab
- **Authentication**: Django REST Framework Token Auth

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Malombe-dev/server-ELD
   cd server

   