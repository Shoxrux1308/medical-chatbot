# 📊 Project Documentation

## System Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (Browser)                 │
│  HTML5 | CSS3 | JavaScript | Responsive Design      │
└──────────────────┬──────────────────────────────────┘
                   │ HTTP/HTTPS
┌──────────────────▼──────────────────────────────────┐
│              Flask Web Server                        │
│  - Routing                                           │
│  - Template Rendering (Jinja2)                       │
│  - Session Management                               │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│          Business Logic Layer                        │
│  - Authentication (Bcrypt)                           │
│  - AI Analysis Engine                                │
│  - Fuzzy Symptom Matching                            │
│  - Disease Prediction Algorithm                      │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│           Data Layer (SQLAlchemy ORM)                │
│  - User Management                                   │
│  - Analysis History                                  │
│  - Medical Database (In-Memory)                      │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
            ┌──────────────┐
            │ SQLite / DB  │
            └──────────────┘
```

## API Specification

### Authentication Endpoints

#### POST /register
Create new user account
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+998901234567",
  "password": "secure_password",
  "confirm_password": "secure_password"
}
```

#### POST /login
Authenticate user
```json
{
  "email": "john@example.com",
  "password": "secure_password"
}
```

#### GET /logout
Clear session and logout

### Analysis Endpoints

#### POST /api/predict
Get medical prediction
```json
{
  "symptoms": ["fever", "cough", "sore_throat"],
  "days": 2
}
```

Response:
```json
{
  "disease1": "Common Cold",
  "disease2": "Influenza",
  "description1": "A viral infection...",
  "description2": "A contagious respiratory...",
  "precautions1": ["Rest well", "Stay hydrated"],
  "precautions2": ["Get antiviral medication"],
  "advice": "This is usually mild...",
  "advice_type": "success",
  "same_prediction": false,
  "related_symptoms": [
    {"value": "loss_of_appetite", "label": "Loss of Appetite"}
  ],
  "confidence": "high"
}
```

#### POST /api/ai-chat
Chat with AI advisor
```json
{
  "message": "I have fever and cough",
  "current_symptoms": ["fever"]
}
```

Response:
```json
{
  "type": "symptoms_found",
  "message": "I found these symptoms",
  "symptoms": ["cough"],
  "symptom_labels": ["Cough"],
  "follow_up": "Would you like to add more?"
}
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name VARCHAR(120) NOT NULL,
  email VARCHAR(120) UNIQUE NOT NULL,
  phone VARCHAR(20),
  password_hash VARCHAR(255) NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Analyses Table
```sql
CREATE TABLE analyses (
  id INTEGER PRIMARY KEY,
  user_id INTEGER NOT NULL FOREIGN KEY,
  symptoms_list VARCHAR(500),
  disease1 VARCHAR(120),
  disease2 VARCHAR(120),
  confidence VARCHAR(20),
  advice TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## AI Algorithm Details

### Symptom Matching
```python
def fuzzy_match(symptom, query):
    return SequenceMatcher(None, 
                          symptom.lower(), 
                          query.lower()).ratio() > 0.7
```

### Disease Scoring
```
Score = (Matching Symptoms / Total Disease Symptoms) * 100
```

### Confidence Levels
- **HIGH**: Score > 70% or same prediction from multiple models
- **MEDIUM**: Score 50-70%
- **LOW**: Score < 50%

## File Structure Detailed

```
medical-chatbot/
│
├── app.py                          # Main Flask application
│   ├── Database models (User, Analysis)
│   ├── Medical database (MEDICAL_DATABASE)
│   ├── Translation system
│   ├── Authentication logic
│   ├── API endpoints
│   └── AI engine
│
├── templates/                      # Jinja2 HTML templates
│   ├── base.html                  # Base template layout
│   ├── index.html                 # Home/landing page
│   ├── login.html                 # Login form
│   ├── register.html              # Registration form
│   ├── chat.html                  # Main analysis interface
│   └── profile.html               # User profile page
│
├── static/                        # Static assets
│   ├── css/
│   │   └── main.css              # Global styles
│   └── js/                        # Optional JavaScript
│
├── instance/                      # Flask instance folder
│   └── medsync.db                # SQLite database
│
├── .env                           # Environment variables
├── .gitignore                     # Git ignore rules
├── requirements.txt               # Python dependencies
├── Procfile                       # Deployment configuration
├── runtime.txt                    # Python version
│
├── README.md                      # Project README
├── DEPLOYMENT.md                  # Deployment guide
└── DOCUMENTATION.md              # This file
```

## Multi-Language Implementation

### Language Structure
```python
translations = {
    'uz': { 'key': 'Uzbek text', ... },
    'en': { 'key': 'English text', ... },
    'ru': { 'key': 'Russian text', ... }
}
```

### Session Management
```python
@app.before_request
def before_request():
    if 'lang' not in session:
        session['lang'] = 'uz'  # Default language
```

### Template Usage
```html
{{ t.key_name }}  <!-- Displays translated text -->
```

### Language Switching
```html
<a href="/set-language/en">English</a>
<a href="/set-language/uz">Uzbek</a>
<a href="/set-language/ru">Russian</a>
```

## Security Implementation

### Password Security
```python
from werkzeug.security import generate_password_hash, check_password_hash

user.password_hash = generate_password_hash(password)
user.check_password(input_password)  # Returns True/False
```

### Session Security
```python
app.config['SESSION_TYPE'] = 'filesystem'
app.config['PERMANENT_SESSION_LIFETIME'] = 2592000  # 30 days
```

### CSRF Protection Ready
```python
# Can be enabled with Flask-WTF
from flask_wtf.csrf import CSRFProtect
csrf = CSRFProtect(app)
```

## Performance Considerations

### Current Implementation
- SQLite database (suitable for MVP)
- In-memory medical database
- Fuzzy matching for symptom recognition
- Simple session storage

### Optimization Strategies
1. **Database**: Migrate to PostgreSQL for production
2. **Caching**: Implement Redis for session storage
3. **Static Files**: Use CDN (CloudFlare, AWS S3)
4. **Compression**: Enable Gzip compression in Nginx
5. **Async Tasks**: Use Celery for heavy computations

## Medical Data Management

### Adding New Symptoms
Edit `MEDICAL_DATABASE['symptoms']`:
```python
{'key': 'new_symptom', 
 'name': 'English Name',
 'name_uz': 'Uzbek Name',
 'name_ru': 'Russian Name'}
```

### Adding New Diseases
Edit `MEDICAL_DATABASE['diseases']`:
```python
{
    'key': 'disease_key',
    'name': 'Disease Name',
    'name_uz': 'Uzbek Name',
    'name_ru': 'Russian Name',
    'symptoms': ['symptom_key1', 'symptom_key2'],
    'description': 'Medical description',
    'precautions': ['Precaution 1', 'Precaution 2'],
    'advice': 'Medical advice',
    'advice_type': 'success|danger'
}
```

## Testing Checklist

- [ ] User Registration (valid/invalid email, password validation)
- [ ] User Login (correct/incorrect credentials)
- [ ] Symptom Selection (single, multiple, search)
- [ ] Disease Prediction (accuracy, confidence levels)
- [ ] AI Chat (symptom recognition, suggestions)
- [ ] Profile Page (history display, user info)
- [ ] Language Switching (all 3 languages)
- [ ] Responsive Design (desktop, tablet, mobile)
- [ ] Error Handling (400, 404, 500 errors)

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Database locked | Restart Flask app |
| Port 5000 in use | `sudo lsof -i :5000` → `kill -9 <PID>` |
| Import errors | `pip install --upgrade -r requirements.txt` |
| 502 Bad Gateway | Check Gunicorn process status |
| Static files not loading | Check `url_for()` paths in templates |

## Future Enhancements

1. **Advanced AI**
   - Integration with ML models (Scikit-learn, TensorFlow)
   - Real doctor verification system
   - Symptom recommendation engine

2. **Features**
   - Video consultation integration
   - Medicine recommendation
   - Appointment booking
   - Insurance verification

3. **Scalability**
   - Microservices architecture
   - Load balancing
   - Caching layer
   - Message queues

4. **Analytics**
   - User behavior tracking
   - Diagnosis accuracy metrics
   - Popular symptoms analysis
   - Regional health trends

## Support & Contact

For technical issues:
- Check README.md
- Review DEPLOYMENT.md
- Open GitHub issue
- Contact development team

---

**Last Updated:** 2026-05-09
**Version:** 1.0.0
**Status:** Production Ready
