# Ajira-Backend
Backend System architecture 
🧩 TECHNOLOGY STACK
Layer	Stack
Language	Python 3.11+
Framework	Django 5.x + Django REST Framework (FBVs)
Auth	SimpleJWT (JWT-based authentication)
Database	PostgreSQL
Async Tasks	Celery + Redis
File Storage	Django default or django-storages (S3)
AI Assistant	OpenAI or local model via REST endpoint
SMS & Email	Africa’s Talking / Twilio / SendGrid

📁 PROJECT STRUCTURE
bash
Copy
Edit
/backend
├── config/                    # Main Django project
│   ├── settings.py
│   ├── urls.py
│   └── celery.py
│
├── apps/
│   ├── users/                 # Auth, roles, employee data
│   ├── attendance/            # Clock-in/out, GPS
│   ├── payroll/               # Salaries, deductions, payslips
│   ├── leave/                 # Leave requests + approval
│   ├── performance/           # KPI, feedback
│   ├── chatbot/               # AI assistant
│   ├── wellness/              # Mood checks, feedback
│   └── notifications/         # SMS + email (via Celery)
│
├── static/ / media/           # Optional for admin / files
├── manage.py
└── requirements.txt
🔐 USERS / AUTH MODULE
model: CustomUser
python
Copy
Edit
class CustomUser(AbstractUser):
    ROLE_CHOICES = [
        ('admin', 'Admin'),
        ('hr', 'HR Manager'),
        ('supervisor', 'Supervisor'),
        ('employee', 'Employee')
    ]
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='employee')
    phone = models.CharField(max_length=15)
    national_id = models.CharField(max_length=25, null=True, blank=True)
views: FBVs
python
Copy
Edit
@api_view(['POST'])
def login_view(request):
    ...

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def me_view(request):
    ...
endpoints
Method	URL	Description
POST	/api/auth/login/	Get access + refresh JWT
GET	/api/auth/me/	Current logged-in user

🕒 ATTENDANCE
model
python
Copy
Edit
class Attendance(models.Model):
    employee = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    check_in = models.DateTimeField()
    check_out = models.DateTimeField(null=True, blank=True)
    gps_location = models.JSONField()
    method = models.CharField(max_length=10)
views
python
Copy
Edit
@api_view(['POST'])
@permission_classes([IsAuthenticated])
def clock_in_view(request):
    ...
endpoints
Method	URL	Description
POST	/api/attendance/clock-in/	Clock in employee
POST	/api/attendance/clock-out/	Clock out employee
GET	/api/attendance/history/	View personal history

🌴 LEAVE MANAGEMENT
models
python
Copy
Edit
class LeaveType(models.Model):
    name = models.CharField(max_length=50)
    max_days = models.IntegerField()

class LeaveRequest(models.Model):
    employee = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    leave_type = models.ForeignKey(LeaveType, on_delete=models.CASCADE)
    start_date = models.DateField()
    end_date = models.DateField()
    status = models.CharField(max_length=10, choices=[('pending', 'Pending'), ...])
    approved_by = models.ForeignKey(CustomUser, null=True, blank=True, on_delete=models.SET_NULL, related_name='approver')
views
python
Copy
Edit
@api_view(['POST'])
@permission_classes([IsAuthenticated])
def request_leave_view(request):
    ...
endpoints
Method	URL	Description
POST	/api/leave/request/	Submit a leave request
GET	/api/leave/history/	View user leave history
PUT	/api/leave/<id>/approve/	HR approves/rejects request

💰 PAYROLL
models
python
Copy
Edit
class Salary(models.Model):
    employee = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    base_salary = models.DecimalField(...)
    allowances = models.DecimalField(...)
    deductions = models.DecimalField(...)
    tax = models.DecimalField(...)
    net_pay = models.DecimalField(...)

class Payslip(models.Model):
    employee = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    month = models.DateField()
    generated_on = models.DateTimeField(auto_now_add=True)
    file = models.FileField(upload_to='payslips/', null=True, blank=True)
views
python
Copy
Edit
@api_view(['POST'])
@permission_classes([IsAdminUser])
def generate_payroll_view(request):
    ...
endpoints
Method	URL	Description
POST	/api/payroll/generate/	Generate monthly payroll
GET	/api/payroll/payslips/	View all payslips

📈 PERFORMANCE REVIEWS
model
python
Copy
Edit
class PerformanceReview(models.Model):
    employee = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    reviewer = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name="reviewed_by")
    date = models.DateField()
    kpi_score = models.FloatField()
    feedback = models.TextField()
    goals = models.JSONField()
endpoints
Method	URL	Description
POST	/api/performance/review/	Submit review
GET	/api/performance/my-reviews/	View your reviews
GET	/api/performance/employee/<id>/	View employee reviews (HR)

🤖 CHATBOT
views
python
Copy
Edit
@api_view(['POST'])
def chatbot_view(request):
    prompt = request.data.get("query")
    ...
endpoints
Method	URL	Description
POST	/api/chatbot/query/	Submit question to AI bot

💬 NOTIFICATIONS (SMS / EMAIL)
tasks.py (Celery)
python
Copy
Edit
@shared_task
def send_sms(phone, message):
    ...

@shared_task
def send_email(to, subject, message):
    ...
endpoints
Method	URL	Description
POST	/api/notifications/send-sms/	Send SMS (async)
POST	/api/notifications/send-email/	Send Email

🧠 WELLNESS & FEEDBACK
endpoints
Method	URL	Description
POST	/api/wellness/mood-check/	Submit mood anonymously
POST	/api/wellness/report-issue/	Submit anonymous complaint
GET	/api/wellness/summary/	HR view of wellness trends

🧪 SYSTEM EXTRAS
Celery for async: used for payroll processing, notifications

OpenAI/LLM integration: chatbot assistant

CORS headers: for frontend access

Role-based decorators: @hr_only, @is_employee

Swagger / ReDoc: for interactive API docs (drf-spectacular)

🔐 SECURITY CONFIG
Feature	Tool / Config
Auth	SimpleJWT
Permissions	IsAuthenticated, IsAdminUser, custom roles
CORS	django-cors-headers
Rate Limiting	throttling classes (DRF)
Audit Logging	Custom middleware / logging

✅ END RESULT
This backend is:

🔌 Modular and plug-and-play

🔐 Role-secured and scalable

🔁 React-ready (Vite-friendly JSON APIs)

⚡ Async-capable via Celery

🤖 AI-enabled with chatbot endpoints
