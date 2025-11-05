
# ELD Log Generator - Backend API

Django REST API for the Electronic Logging Device (ELD) application that calculates HOS-compliant routes and manages driver logs.

## 🚀 Features

- **Route Calculation**: Generates HOS-compliant routes with automatic stops
- **Fuel Stop Planning**: Adds fuel stops every 1,000 miles
- **Rest Break Calculation**: Inserts mandatory rest breaks per FMCSA regulations
- **Log Management**: Stores and retrieves driver daily logs
- **PDF Generation**: Creates printable ELD log sheets
- **CORS Support**: Enables cross-origin requests from frontend

## 📋 Prerequisites

- Python 3.8 or higher
- pip (Python package manager)
- Virtual environment (recommended)

## 🛠️ Installation

### 1. Clone the repository
```bash
git clone https://github.com/Malombe-dev/server-ELD
cd backend
```

### 2. Create virtual environment
```bash
# On Windows
python -m venv venv
venv\Scripts\activate

# On macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the root directory:
```env
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:3000
DATABASE_URL=sqlite:///db.sqlite3
```

For production:
```env
SECRET_KEY=your-production-secret-key
DEBUG=False
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
CORS_ALLOWED_ORIGINS=https://your-frontend-url.vercel.app
DATABASE_URL=postgresql://user:password@host:port/database
```

### 5. Run migrations
```bash
python manage.py migrate
```

### 6. Create superuser (optional)
```bash
python manage.py createsuperuser
```

### 7. Start development server
```bash
python manage.py runserver
```

The API will be available at `http://localhost:8000`

## 📦 Dependencies

```txt
Django==4.2.7
djangorestframework==3.14.0
django-cors-headers==4.3.0
python-decouple==3.8
requests==2.31.0
reportlab==4.0.7  # For PDF generation
```

## 🏗️ Project Structure

```
backend/
├── eld_backend/
│   ├── __init__.py
│   ├── settings.py          # Django settings
│   ├── urls.py              # Main URL configuration
│   ├── wsgi.py
│   └── asgi.py
├── api/
│   ├── __init__.py
│   ├── models.py            # Database models
│   ├── serializers.py       # DRF serializers
│   ├── views.py             # API endpoints
│   ├── urls.py              # API URL routes
│   └── utils.py             # Helper functions
├── manage.py
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

## 🔌 API Endpoints

### 1. Calculate Route
Calculate HOS-compliant route with stops.

**Endpoint**: `POST /api/calculate-route/`

**Request Body**:
```json
{
  "origin": "New York, NY",
  "destination": "Los Angeles, CA",
  "waypoints": ["Chicago, IL"],
  "current_cycle_hours": 10.5,
  "driver_name": "John Doe",
  "carrier_name": "ABC Transport",
  "start_time": "08:00"
}
```

**Response**:
```json
{
  "stops": [
    {
      "type": "start",
      "location": "New York, NY",
      "time": "08:00 AM",
      "title": "Start Location",
      "notes": "Trip start - Pre-trip inspection"
    },
    {
      "type": "pickup",
      "location": "Chicago, IL",
      "time": "11:30 AM",
      "duration": "1h",
      "title": "Pickup Location",
      "notes": "Load cargo - 1 hour"
    },
    {
      "type": "fuel",
      "location": "Calculated location",
      "time": "03:00 PM",
      "duration": "0.5h",
      "title": "Fuel Stop",
      "notes": "Refueling required"
    },
    {
      "type": "rest",
      "location": "Rest area",
      "time": "08:00 PM",
      "duration": "10h",
      "title": "Mandatory Rest Break",
      "notes": "10-hour break required"
    },
    {
      "type": "dropoff",
      "location": "Los Angeles, CA",
      "time": "06:00 PM (next day)",
      "duration": "1h",
      "title": "Dropoff Location",
      "notes": "Unload cargo"
    }
  ],
  "totalDistance": "2789 miles",
  "totalDuration": "45h 30m",
  "drivingTime": "32h 15m",
  "restTime": "13h 15m",
  "fuelStops": 2,
  "compliance": {
    "isCompliant": true,
    "violations": []
  }
}
```

### 2. Save Daily Log
Save a driver's daily log.

**Endpoint**: `POST /api/save-log/`

**Request Body**:
```json
{
  "date": "11/5/2025",
  "driver": "John Doe",
  "carrier": "ABC Transport",
  "segments": [
    {
      "status": 0,
      "duration": 2.5,
      "startTime": "2025-11-05T06:00:00",
      "endTime": "2025-11-05T08:30:00",
      "start": 6,
      "end": 8.5
    }
  ],
  "summary": {
    "offDuty": 12.0,
    "sleeper": 8.0,
    "driving": 3.5,
    "onDuty": 0.5
  },
  "totalMiles": 192,
  "remarks": "Drove 3.5h. Locations: New York, Chicago.",
  "tripData": {
    "vehicleNumber": "V-1234",
    "trailerNumber": "T-5678"
  }
}
```

**Response**:
```json
{
  "id": 1,
  "success": true,
  "message": "Log saved successfully"
}
```

### 3. Get Driver Logs
Retrieve all logs for a driver.

**Endpoint**: `GET /api/driver-logs/`

**Query Parameters**: 
- `driver_name` (optional): Filter by driver name
- `date_from` (optional): Start date filter
- `date_to` (optional): End date filter

**Response**:
```json
[
  {
    "id": 1,
    "date": "11/5/2025",
    "driver": "John Doe",
    "totalMiles": 192,
    "summary": {...},
    "segments": [...]
  }
]
```

### 4. Download Logs as PDF
Generate and download PDF of logs.

**Endpoint**: `POST /api/download-logs-pdf/`

**Request Body**:
```json
{
  "logs": [
    {
      "date": "11/5/2025",
      "driver": "John Doe",
      "segments": [...],
      "summary": {...}
    }
  ]
}
```

**Response**: PDF file download

## 💾 Database Models

### DriverLog Model
```python
class DriverLog(models.Model):
    driver_name = models.CharField(max_length=255)
    date = models.DateField()
    vehicle_number = models.CharField(max_length=50)
    trailer_number = models.CharField(max_length=50)
    total_miles = models.IntegerField()
    off_duty_hours = models.FloatField()
    sleeper_hours = models.FloatField()
    driving_hours = models.FloatField()
    on_duty_hours = models.FloatField()
    remarks = models.TextField()
    segments = models.JSONField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

## 🧮 Route Calculation Algorithm

### HOS Compliance Rules
```python
DRIVING_LIMIT_PER_DAY = 11  # hours
ON_DUTY_LIMIT_PER_DAY = 14  # hours
CYCLE_LIMIT = 70  # hours over 8 days
MANDATORY_BREAK_AFTER = 8  # hours of driving
BREAK_DURATION = 10  # hours
FUEL_STOP_INTERVAL = 1000  # miles
PICKUP_DROPOFF_DURATION = 1  # hour
```

### Algorithm Steps
1. **Geocode locations** using external geocoding service
2. **Calculate distance** between points
3. **Determine driving time** (average 55 mph)
4. **Insert fuel stops** every 1,000 miles (30 min each)
5. **Add rest breaks** after 8 hours of driving (10 hours)
6. **Include pickup/dropoff time** (1 hour each)
7. **Validate HOS compliance**
8. **Generate timeline** with all stops

### Example Calculation
```python
def calculate_route(origin, destination, waypoints, current_cycle):
    total_distance = get_distance(origin, destination, waypoints)
    driving_hours = total_distance / 55  # Average speed
    
    stops = []
    current_time = start_time
    distance_since_fuel = 0
    driving_since_break = 0
    
    # Add start
    stops.append(create_stop('start', origin, current_time))
    
    # Process waypoints
    for waypoint in waypoints:
        segment_distance = get_distance(current_location, waypoint)
        segment_time = segment_distance / 55
        
        # Check for fuel stop
        if distance_since_fuel + segment_distance >= 1000:
            fuel_location = calculate_fuel_point()
            stops.append(create_stop('fuel', fuel_location, current_time))
            current_time += 0.5  # 30 min fuel stop
            distance_since_fuel = 0
        
        # Check for rest break
        if driving_since_break + segment_time >= 8:
            stops.append(create_stop('rest', current_location, current_time))
            current_time += 10  # 10 hour break
            driving_since_break = 0
        
        stops.append(create_stop('pickup', waypoint, current_time))
        current_time += 1  # Pickup time
    
    # Add final destination
    stops.append(create_stop('dropoff', destination, current_time))
    
    return stops
```

## 🔧 Configuration

### settings.py Key Settings

```python
# CORS Configuration
CORS_ALLOW_ALL_ORIGINS = False
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "https://your-frontend-url.vercel.app"
]

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
    ],
}

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

## 🧪 Testing

### Run Tests
```bash
python manage.py test
```

### Manual API Testing

Using cURL:
```bash
# Calculate Route
curl -X POST http://localhost:8000/api/calculate-route/ \
  -H "Content-Type: application/json" \
  -d '{
    "origin": "New York, NY",
    "destination": "Los Angeles, CA",
    "waypoints": ["Chicago, IL"],
    "current_cycle_hours": 10
  }'
```

Using Postman:
1. Import the API collection (if provided)
2. Set base URL to `http://localhost:8000`
3. Test each endpoint

## 🚀 Deployment

### Deploy to Railway

1. **Install Railway CLI**
```bash
npm install -g @railway/cli
```

2. **Initialize Railway**
```bash
railway login
railway init
```

3. **Add PostgreSQL**
```bash
railway add
# Select PostgreSQL
```

4. **Deploy**
```bash
railway up
```

5. **Set Environment Variables** in Railway dashboard

### Deploy to Heroku

1. **Create Heroku app**
```bash
heroku create your-app-name
```

2. **Add PostgreSQL**
```bash
heroku addons:create heroku-postgresql:hobby-dev
```

3. **Set environment variables**
```bash
heroku config:set SECRET_KEY=your-secret-key
heroku config:set DEBUG=False
```

4. **Deploy**
```bash
git push heroku main
```

5. **Run migrations**
```bash
heroku run python manage.py migrate
```

### Deploy to PythonAnywhere

1. Create account at [PythonAnywhere.com](https://www.pythonanywhere.com)
2. Upload code via Git or file upload
3. Set up virtual environment
4. Configure WSGI file
5. Reload web app

## 📊 Performance Optimization

- **Database Indexing**: Add indexes to frequently queried fields
- **Caching**: Implement Redis for geocoding results
- **Rate Limiting**: Add throttling to prevent abuse
- **Query Optimization**: Use select_related() and prefetch_related()

## 🔒 Security Checklist

- [ ] Change SECRET_KEY in production
- [ ] Set DEBUG=False in production
- [ ] Configure proper ALLOWED_HOSTS
- [ ] Use environment variables for sensitive data
- [ ] Enable HTTPS in production
- [ ] Implement rate limiting
- [ ] Add authentication/authorization
- [ ] Validate all input data
- [ ] Use parameterized queries
- [ ] Keep dependencies updated

## 🐛 Troubleshooting

### Common Issues

**CORS errors**
```python
# Check CORS settings in settings.py
CORS_ALLOWED_ORIGINS = ['http://localhost:3000']
```

**Database errors**
```bash
# Reset database
python manage.py flush
python manage.py migrate
```

**Import errors**
```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

## 📚 Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [FMCSA Regulations](https://www.fmcsa.dot.gov/)
- [Python Best Practices](https://docs.python-guide.org/)

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

## 📝 License

This project is licensed under the MIT License.

## 👥 Support

For issues or questions:
- Open an issue on GitHub
- Email: [your-email@example.com]

---

**Built with Django REST Framework for HOS compliance**