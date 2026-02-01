# Webhook Implementation - Code Files

This folder contains all the code files needed to implement webhooks between your HRMS and Learning Project.

## 📁 Folder Structure

```
webhook-code-files/
├── hrms/                               # HRMS (Sender) files
│   ├── WebhookConfig.java             # Configuration class
│   ├── WebhookPayload.java            # DTO for webhook data
│   ├── WebhookService.java            # Service to send webhooks
│   ├── RestTemplateConfig.java        # HTTP client configuration
│   ├── AsyncConfig.java               # Enable async processing
│   ├── application.properties         # Configuration properties
│   └── NotificationMessageServiceExample.java  # Example integration
│
├── learning/                           # Learning Project (Receiver) files
│   ├── WebhookNotification.java       # Entity class
│   ├── WebhookPayloadDTO.java         # DTO for receiving data
│   ├── WebhookNotificationRepository.java  # Database repository
│   ├── WebhookService.java            # Service to process webhooks
│   ├── WebhookController.java         # REST endpoints
│   └── schema.sql                     # Database schema
│
├── testing-guide.sh                    # Testing commands
└── README.md                          # This file
```

## 🚀 Getting Started

### Step 1: HRMS Setup (Sender)

1. **Copy Java files to your HRMS project:**
   - Copy all files from `hrms/` folder to your HRMS project
   - Place them in the appropriate packages:
     - `WebhookConfig.java` → `com.yourcompany.hrms.config`
     - `WebhookPayload.java` → `com.yourcompany.hrms.dto`
     - `WebhookService.java` → `com.yourcompany.hrms.service`
     - `RestTemplateConfig.java` → `com.yourcompany.hrms.config`
     - `AsyncConfig.java` → `com.yourcompany.hrms.config`

2. **Add configuration to application.properties:**
   - Copy contents from `hrms/application.properties`
   - Add to your existing HRMS `application.properties` file

3. **Modify your NotificationMessageService:**
   - Open `NotificationMessageServiceExample.java` for reference
   - Add webhook call after creating notifications
   - See the "NEW CODE: Send Webhook" section in the example

### Step 2: Learning Project Setup (Receiver)

1. **Copy Java files to your Learning Project:**
   - Copy all files from `learning/` folder to your Learning Project
   - Place them in the appropriate packages:
     - `WebhookNotification.java` → `com.yourcompany.learning.entity`
     - `WebhookPayloadDTO.java` → `com.yourcompany.learning.dto`
     - `WebhookNotificationRepository.java` → `com.yourcompany.learning.repository`
     - `WebhookService.java` → `com.yourcompany.learning.service`
     - `WebhookController.java` → `com.yourcompany.learning.controller`

2. **Create database table:**
   - Run the SQL from `learning/schema.sql` in your database
   - This creates the `webhook_notifications` table

### Step 3: Testing

1. **Start both applications:**
   ```bash
   # Terminal 1: Start HRMS
   cd /path/to/hrms
   ./mvnw spring-boot:run

   # Terminal 2: Start Learning Project
   cd /path/to/learning-project
   ./mvnw spring-boot:run
   ```

2. **Run tests:**
   - Use commands from `testing-guide.sh`
   - Or follow the testing steps in the main documentation

## 📝 Package Names

**Important:** Replace the package names in all Java files with your actual package names:

- Change `com.yourcompany.hrms` to your HRMS base package
- Change `com.yourcompany.learning` to your Learning Project base package

## 🔍 What Each File Does

### HRMS Files (Sender)

| File | Purpose |
|------|---------|
| `WebhookConfig.java` | Stores webhook configuration (URL, enabled flag) |
| `WebhookPayload.java` | Defines the structure of data sent in webhooks |
| `WebhookService.java` | Handles sending HTTP POST requests to Learning Project |
| `RestTemplateConfig.java` | Configures HTTP client for making requests |
| `AsyncConfig.java` | Enables asynchronous processing (non-blocking) |
| `application.properties` | Configuration values for webhook URL |
| `NotificationMessageServiceExample.java` | Shows how to integrate webhook into your service |

### Learning Project Files (Receiver)

| File | Purpose |
|------|---------|
| `WebhookNotification.java` | Database entity for storing webhook data |
| `WebhookPayloadDTO.java` | Defines structure for receiving webhook data |
| `WebhookNotificationRepository.java` | Database operations for webhooks |
| `WebhookService.java` | Business logic for processing webhooks |
| `WebhookController.java` | REST endpoints for receiving webhooks |
| `schema.sql` | Database table creation script |

## ✅ Verification Checklist

Before testing, make sure:

- [ ] All HRMS files are in correct packages
- [ ] All Learning Project files are in correct packages
- [ ] Package names are updated in all files
- [ ] Database table is created
- [ ] Configuration is added to application.properties
- [ ] Both applications can start without errors
- [ ] HRMS is running on port 8080
- [ ] Learning Project is running on port 8081

## 🧪 Quick Test

After setup, run this quick test:

```bash
# Test webhook endpoint
curl -X POST http://localhost:8081/api/webhooks/notifications \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "notification.created",
    "timestamp": "2024-02-01T10:30:00",
    "data": {
      "notificationId": 123,
      "message": "Test webhook",
      "userId": "user001",
      "type": "REAL_TIME",
      "createdAt": "2024-02-01T10:30:00"
    }
  }'
```

Expected response:
```json
{
  "success": true,
  "message": "Webhook received and processed",
  "notificationId": 1
}
```

## 🐛 Troubleshooting

**Problem:** Compilation errors
- **Solution:** Make sure all imports are correct and package names match your project structure

**Problem:** Connection refused
- **Solution:** Verify Learning Project is running on port 8081

**Problem:** Webhook not triggered
- **Solution:** Check `webhook.enabled=true` in HRMS configuration

**Problem:** Database errors
- **Solution:** Verify database table is created correctly

## 📚 Additional Resources

- Main documentation: `Webhook-Implementation-Guide.docx`
- Testing guide: `testing-guide.sh`
- Original markdown: `webhook-implementation-guide.md`

## 🎯 Next Steps

Once basic implementation is working, consider adding:

1. **Security:** HMAC signature verification
2. **Retry logic:** Exponential backoff for failed webhooks
3. **Monitoring:** Track webhook success/failure rates
4. **Webhook history:** Log all webhook attempts
5. **Multiple event types:** Support different notification types

## 💡 Tips

- Start simple: Get basic webhook working first
- Test thoroughly: Use the testing guide commands
- Add logging: Use logger statements to debug issues
- Monitor performance: Check if webhooks are being sent asynchronously
- Handle errors gracefully: Webhook failures shouldn't break main flow

## ❓ Need Help?

If you encounter issues:

1. Check application logs for error messages
2. Verify all configuration is correct
3. Test each component individually
4. Use the testing guide to isolate problems
5. Review the main documentation for detailed explanations

---

**Good luck with your webhook implementation! 🚀**
