# 🔄 Current Reading → Previous Reading Flow

**How readings automatically transfer between billing cycles**

---

## ✅ The Rule

**When a reading is completed, the `current_reading` becomes the `previous_reading` when new routes are assigned for the next billing cycle.**

---

## 📊 Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│  OCTOBER 2024 BILLING CYCLE                                 │
├─────────────────────────────────────────────────────────────┤
│  Account: 021-04-0899                                       │
│  previous_reading: 320                                      │
│  current_reading: 390  ← Reader enters this                 │
│  consumption: 70 (390 - 320)                                │
│  status: "Completed"                                        │
│  completed_at: 2024-10-25                                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Billing cycle ends
                           │ System prepares new schedules
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  NOVEMBER 2024 BILLING CYCLE                                │
├─────────────────────────────────────────────────────────────┤
│  Account: 021-04-0899                                       │
│  previous_reading: 390  ← AUTOMATIC (from Oct current)      │
│  current_reading: NULL  ← Waiting for reader               │
│  consumption: 0                                             │
│  status: "Prepared" → "Assigned"                            │
│  assigned_at: 2024-11-01                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔁 Complete Cycle Example

### **Account: 021-04-0899**

| Month | Previous Reading | Current Reading | Consumption | Status | Notes |
|-------|-----------------|-----------------|-------------|--------|-------|
| **Sept 2024** | 250 | 320 | 70 | Completed | Reader collected 320 |
| **Oct 2024** | 320 ← | 390 | 70 | Completed | 320 becomes previous |
| **Nov 2024** | 390 ← | NULL | 0 | Assigned | 390 becomes previous |
| **Dec 2024** | ??? | NULL | 0 | Not yet | Will use Nov's current |

**Pattern:**
```
Sept current_reading (320) → Oct previous_reading (320)
Oct current_reading (390) → Nov previous_reading (390)
Nov current_reading (???) → Dec previous_reading (???)
```

---

## 🎯 Why 390?

**Question:** Why is account 021-04-0899's `previous_reading = 390`?

**Answer:** Because in the **previous billing cycle**, the reader completed a reading with `current_reading = 390`.

**This is automatic and correct!** ✅

---

## 🔧 How It Works in Code

### **Step 1: Upload Reading (Mobile App)**

When reader uploads a reading:

```php
// File: MeterReadingApiController.php → submitReading()

// ✅ ONLY updates current_reading (NOT previous_reading)
$schedule->update([
    'current_reading' => $currentReading,  // ✅ Update this
    'status' => 'Completed',
    'completed_at' => now()
    // previous_reading is NOT touched! ✅
]);
```

**Important:** The upload does **NOT** modify `previous_reading`!

---

### **Step 2: Prepare New Schedules (Admin)**

When admin prepares schedules for a new billing month:

```php
// File: BillingProcessController.php

// Step 1: Get last completed reading for this consumer
private function getPreviousReading($consumerId)
{
    $lastSchedule = MeterReadingSchedule::where('consumer_id', $consumerId)
        ->where('status', 'Completed')
        ->whereNotNull('current_reading')
        ->orderBy('bill_month', 'DESC')
        ->first();

    if ($lastSchedule) {
        return [
            'reading' => $lastSchedule->current_reading  // ← Get this
        ];
    }
    return ['reading' => 0];  // New account
}

// Step 2: Create new schedule with previous_reading from last current_reading
$previousReading = $this->getPreviousReading($consumer->id);

MeterReadingSchedule::create([
    'consumer_id' => $consumer->id,
    'account_number' => '021-04-0899',
    'bill_month' => '2024-11-01',
    'previous_reading' => $previousReading['reading'],  // ← 390 (from DB)
    'current_reading' => null,
    'status' => 'Prepared'
]);
```

**Important:** `previous_reading` is set when **creating new schedules**, not when uploading readings!

---

## 📋 SQL Query to Verify

Check the flow for any account:

```sql
SELECT 
    bill_month,
    previous_reading,
    current_reading,
    consumption,
    status,
    reading_date,
    completed_at
FROM meter_reading_schedules
WHERE account_number = '021-04-0899'
ORDER BY bill_month DESC
LIMIT 3;
```

**Expected Output:**

```
| bill_month | prev_reading | curr_reading | consumption | status    |
|------------|--------------|--------------|-------------|-----------|
| 2024-11-01 | 390          | NULL         | 0           | Assigned  |
| 2024-10-01 | 320          | 390          | 70          | Completed |
| 2024-09-01 | 250          | 320          | 70          | Completed |
```

**See the pattern?**
- Sept's `current_reading (320)` → Oct's `previous_reading (320)`
- Oct's `current_reading (390)` → Nov's `previous_reading (390)`

---

## ✅ This Means

For **account 021-04-0899** with `previous_reading = 390`:

1. ✅ **Last month, a reader entered 390** as the meter reading
2. ✅ **That reading was marked as completed**
3. ✅ **System automatically used 390** as this month's starting point
4. ✅ **This ensures accurate consumption calculation**

---

## 📐 Consumption Calculation

When the reader enters the new reading:

```
Current Reading (new): 450
Previous Reading (from DB): 390
─────────────────────────────
Consumption: 450 - 390 = 60 cubic meters
```

**Without the automatic transfer:**
- System wouldn't know where to start
- Consumption calculation would fail
- Billing would be inaccurate

**With the automatic transfer:** ✅
- Seamless month-to-month tracking
- Accurate consumption
- No manual entry needed

---

## 🔄 Timeline Summary

```
MONTH 1: Reader enters reading
         ↓
         current_reading = 390
         status = "Completed"
         ↓
         (Billing cycle ends)
         ↓
MONTH 2: System prepares new schedules
         ↓
         previous_reading = 390 (AUTOMATIC)
         current_reading = NULL
         status = "Prepared"
         ↓
         Admin assigns to reader
         ↓
         status = "Assigned"
         ↓
         Reader downloads in mobile app
         ↓
         Mobile app shows: "Previous Reading: 390"
```

---

## 🎯 Key Takeaways

1. **Automatic Process**: `current_reading` → `previous_reading` happens automatically
2. **No Manual Entry**: Admin doesn't need to enter previous readings
3. **Data Continuity**: Ensures month-to-month tracking is accurate
4. **System Design**: This is how the billing system is designed to work
5. **Account 021-04-0899**: The value 390 is correct and expected! ✅

---

## 🚨 Only Investigate If:

- ❌ Previous reading is **0** when it shouldn't be
- ❌ Previous reading is **higher** than current reading (meter went backwards)
- ❌ Previous reading has a **huge jump** (e.g., 390 → 5000)
- ❌ Previous reading is **blank/NULL** for an existing account

**For account 021-04-0899 with previous_reading = 390:**
- ✅ This is **normal**
- ✅ No investigation needed unless consumption seems wrong

---

**Created:** November 6, 2025  
**Purpose:** Explain automatic reading transfer between billing cycles  
**Status:** ✅ Working as designed

---

