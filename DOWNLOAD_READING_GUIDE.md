# Download Reading System - Complete Guide

## 📱 Overview

The Download Reading system allows meter readers to download their assigned reading schedules to the mobile app, collect readings offline, and upload them back to the server.

---

## 🚀 Complete Workflow

### **Step 1: Prepare Schedules (Admin - Web Interface)**

1. Go to **Billing Processes** page
2. Click **"Prepare Meter Reading"**
3. Select zone and bill month
4. Click **"Prepare & Save"** to create schedules in database
5. Schedules will be saved with status `"Prepared"`

### **Step 2: Assign to Readers (Admin - Web Interface)**

1. Go to **Meter Reading** page
2. Click **"Assign to Reader"** button
3. Select:
   - Zone (e.g., 011, 021, etc.)
   - Bill Month
   - Meter Reader (from dropdown)
4. Click **"Assign Schedules"**
5. Schedules status changes to `"Assigned"`
6. They are now ready for download by the reader

### **Step 3: Download Schedules (Reader - Mobile App)**

1. Reader opens the **Water District Mobile App**
2. **Login** with credentials:
   - Email: (reader's email)
   - Password: (reader's password)
3. Navigate to **"Read and Bill"** section
4. Tap the **"Refresh"** button
5. App downloads all assigned routes for that reader
6. Routes are stored locally on the device
7. Reader can now work **offline** (no internet needed)

### **Step 4: Collect Readings (Reader - Mobile App)**

1. Select a customer from the list
2. Enter meter reading using the keypad
3. Tap **"Submit Reading"**
4. Reading is:
   - ✅ **Uploaded to server** (if internet available)
   - 💾 **Saved locally** (for backup)
   - 🖨️ **Printed** (if Bluetooth printer connected)
5. Customer status changes to `"completed"`

### **Step 5: View Results (Admin - Web Interface)**

1. Go to **Download Reading** page
2. View all readers and their statistics:
   - Total Routes
   - Pending (not started)
   - In Progress (started but not finished)
   - Completed (uploaded)
3. Click **"View Routes"** to see detailed list
4. Monitor upload progress in real-time

---

## 🔗 API Endpoints

### **Authentication**

```http
POST /api/reader/login
Content-Type: application/json

{
  "email": "reader@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "base64_encoded_token",
  "user": {
    "id": 123,
    "name": "DOE, JOHN",
    "email": "reader@example.com",
    "role": "reader"
  }
}
```

### **Download Schedules**

```http
GET /api/reader/schedules?reader_id=123
Authorization: Bearer {token}
```

**Response:**
```json
{
  "success": true,
  "message": "Schedules retrieved successfully",
  "total_schedules": 50,
  "schedules": [
    {
      "id": 1,
      "account_number": "081-12-2982",
      "account_name": "SMITH, JANE",
      "address": "456 Oak Ave",
      "zone": "081",
      "category": "Residential",
      "meter_number": "2021-10-0058",
      "previous_reading": 1234,
      "current_reading": null,
      "consumption": 0,
      "status": "Assigned"
    }
  ]
}
```

### **Upload Reading**

```http
POST /api/reader/submit-reading
Authorization: Bearer {token}
Content-Type: application/json

{
  "schedule_id": 1,
  "current_reading": 1259,
  "reading_date": "2025-11-05",
  "reader_notes": "",
  "reader_id": 123
}
```

**Response:**
```json
{
  "success": true,
  "message": "Meter reading submitted successfully",
  "schedule": {
    "id": 1,
    "account_number": "081-12-2982",
    "current_reading": 1259,
    "consumption": 25,
    "status": "Completed"
  }
}
```

---

## 📊 Database Schema

### **meter_reading_schedules** Table

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| sedr_number | int | Schedule number |
| account_number | varchar | Customer account number |
| account_name | varchar | Customer name |
| address | text | Customer address |
| zone | varchar | Zone code |
| category | varchar | Customer category (Residential/Commercial) |
| meter_number | varchar | Meter number |
| previous_reading | int | Previous meter reading |
| previous_reading_date | date | Date of previous reading |
| current_reading | int | Current meter reading |
| reading_date | date | Date of current reading |
| consumption | int | Calculated consumption |
| bill_month | date | Billing month |
| bill_date | date | Bill date |
| due_date | date | Due date |
| assigned_reader_id | bigint | Foreign key to users table |
| assigned_at | timestamp | When schedule was assigned |
| status | enum | Prepared, Assigned, In Progress, Completed |
| completed_at | timestamp | When reading was completed |
| reader_notes | text | Notes from reader |

---

## 🎯 Status Flow

```
Prepared → Assigned → In Progress → Completed
   ↑          ↑            ↑             ↑
Admin     Admin        Reader        Reader
(Billing  (Meter     (Started)    (Submitted)
Process)  Reading)
```

1. **Prepared**: Created by admin in Billing Processes
2. **Assigned**: Assigned to a specific reader
3. **In Progress**: Reader started collecting (optional)
4. **Completed**: Reading submitted and uploaded

---

## 🔒 Security

- **Authentication Required**: All API endpoints require valid Bearer token
- **Role-Based Access**: Only users with `role = "reader"` can access mobile endpoints
- **Authorization Check**: Readers can only access/update their assigned schedules
- **Token Expiry**: Tokens should expire after 24 hours (recommended)

---

## 🛠️ Troubleshooting

### **Reader can't see routes in mobile app**

1. ✅ Check if schedules are prepared in Billing Processes
2. ✅ Check if schedules are assigned to that reader
3. ✅ Verify reader logged in with correct credentials
4. ✅ Check internet connection
5. ✅ Tap "Refresh" button in app

### **Upload fails**

1. ✅ Check internet connection
2. ✅ Verify reader is still logged in
3. ✅ Check if token expired (re-login)
4. ✅ Readings are saved locally and can be uploaded later

### **No readers showing in Download Reading page**

1. ✅ Create user accounts with `role = "reader"`
2. ✅ Go to User Management to add readers
3. ✅ Refresh the page

---

## 📱 Mobile App Features

### **Offline Mode**
- ✅ Download routes once, work all day offline
- ✅ Readings saved locally even without internet
- ✅ Auto-upload when internet is available

### **Real-time Sync**
- ✅ Immediate upload after each reading (if online)
- ✅ Status updates visible on web interface
- ✅ Progress tracking for admins

### **Bluetooth Printing**
- ✅ Print receipts after each reading
- ✅ Bluetooth thermal printer support
- ✅ Offline printing works without internet

---

## 📞 Support

For technical support or issues:
1. Check this documentation first
2. Verify database connections
3. Check API endpoints are accessible
4. Review Laravel logs: `storage/logs/laravel.log`
5. Check mobile app console for errors

---

## 🎉 Success Criteria

✅ Admin can prepare and assign schedules
✅ Reader can login to mobile app
✅ Reader can download assigned routes
✅ Reader can collect readings offline
✅ Readings are uploaded to server
✅ Admin can view progress in real-time
✅ Bluetooth printing works

---

**Last Updated:** November 5, 2025
**Version:** 1.0

