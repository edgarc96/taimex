# 🚀 Backend Deployment Instructions - SO Draft System

## ❌ Current Problem

The "My Drafts" functionality is not working because the backend XML files are not compiled on the Pick/BASIC server.

**Error Symptoms:**
- Status: 0 when calling `GET_SO_TMP.XML`
- Request times out after 10 seconds
- No response text received

---

## 📋 Files That Need to be Deployed

### 1. **GET_SO_TMP.XML** ✅ Ready
**Location:** `/Users/edgarcabrera/Documents/TAIMEX2/GET_SO_TMP.XML`
**Purpose:** Retrieves all saved drafts for the current user from TMP database
**Expected Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<tmp_list>
  <temp_so>
    <temp_id>so*tmp*ec*1728000000000</temp_id>
    <so_number>SO-12345</so_number>
    <customer_code>CUST001</customer_code>
    <shipto_code>SHIP001</shipto_code>
    <save_date>10/03/2025</save_date>
    <save_time>15:30:45</save_time>
    <num_items>3</num_items>
  </temp_so>
</tmp_list>
```

### 2. **SAVE_SO_TMP.XML** ✅ Ready
**Location:** `/Users/edgarcabrera/Documents/TAIMEX2/SAVE_SO_TMP.XML`
**Purpose:** Saves SO draft to TMP database
**Expected Response:**
```xml
<?xml version="1.0"?>
<response>
  <success>true</success>
  <message>SO saved to TMP database for later</message>
  <temp_id>so*tmp*ec*1728000000000</temp_id>
</response>
```

### 3. **DELETE_SO_TMP.XML** ⚠️ May be needed
**Location:** `/Users/edgarcabrera/Documents/TAIMEX2/DELETE_SO_TMP.XML`
**Purpose:** Deletes a draft from TMP database

---

## 🔧 Deployment Steps

### Step 1: Access Pick/BASIC Server
```bash
# SSH or telnet into your Pick/BASIC server
ssh user@your-picbasic-server
# or
telnet your-picbasic-server
```

### Step 2: Navigate to BP (Basic Programs) Directory
```
:CD BP
```

### Step 3: Upload Files
Transfer the XML files to the BP directory on the server. Methods:
- FTP
- SCP
- Copy/paste via terminal editor
- PicLan file transfer utility

### Step 4: Compile Each File
```
:COMPILE BP GET_SO_TMP.XML
:COMPILE BP SAVE_SO_TMP.XML
:COMPILE BP DELETE_SO_TMP.XML
```

**Expected Output:**
```
GET_SO_TMP.XML
Compiling...
0 Errors detected
0 Warnings
```

### Step 5: Verify Compilation
```
:LIST BP GET_SO_TMP.XML
:LIST BP SAVE_SO_TMP.XML
```

Should show the compiled code without errors.

### Step 6: Test from Browser
Open browser console and run:
```javascript
fetch('GET_SO_TMP.XML?user_initials=EC')
  .then(r => r.text())
  .then(console.log);
```

**Expected:** XML response with `<tmp_list>` root element
**Not Expected:** Status 0, timeout, or empty response

---

## 🧪 Testing Checklist

After deployment, test in this order:

### ✅ Test 1: GET_SO_TMP.XML
1. Open SO_EDIT.HTM
2. Click "My Drafts" button
3. Check console logs:
   ```
   📡 Fetching drafts from: GET_SO_TMP.XML?user_initials=EC
   📥 Response received - Status: 200  ← Should be 200, not 0
   📄 Response text: <?xml version="1.0"...
   ```

### ✅ Test 2: SAVE_SO_TMP.XML
1. Create a new SO with line items
2. Click "Continue" → "Save for Later"
3. Check console logs:
   ```
   💾 Saving draft with temp_id: so*tmp*ec*...
   📦 Preparing data for TMP save...
   📊 Extracted line items: ...
   🚀 Sending save request to SAVE_SO_TMP.XML...
   📥 SAVE Response received - Status: 200  ← Should be 200
   ✅ Save result: {success: true, ...}
   ```

### ✅ Test 3: Load Saved Draft
1. Click "My Drafts" again
2. Should see the saved draft in the list
3. Click on it to load

---

## 🔍 Troubleshooting

### Problem: Still getting Status: 0
**Possible Causes:**
1. Files not uploaded to correct directory
2. Compilation errors (check compile output)
3. TMP database doesn't exist
4. XMLDATA file doesn't exist
5. Permissions issue

**Solution:**
```
:CREATE-FILE TMP 1 101
:CREATE-FILE XMLDATA 1 101
```

### Problem: Compilation Errors
**Common Issues:**
- Missing `END` statements
- Incorrect field syntax
- Missing INCLUDE files

**Check:**
```
:COMPILE BP GET_SO_TMP.XML (V
```
The `(V` option shows verbose output.

### Problem: Empty Response
**Possible Causes:**
1. TMP database is empty (no drafts saved yet)
2. User filter not matching

**Test:**
```
:LIST TMP WITH @ID LIKE "so*..."
```
Should show saved draft records.

---

## 📊 Database Structure

### TMP File Structure
**Record Key Format:** `so*tmp*<initials>*<timestamp>`
**Example:** `so*tmp*ec*1728000000000`

**Field Mapping:**
- Field 1: SO Number
- Field 2: Ship To Code (Customer Code)
- Field 3: Bill To Code
- Field 4: Ship Via
- Field 5: Timestamp (date\time with XSM separator)
- Field 11: Terms
- Field 25: Header Notes
- Field 27: Email
- Field 31: Line positions (multivalue)
- Field 32: Part numbers (multivalue, pipe-separated in frontend)
- Field 33: Descriptions (multivalue)
- Field 35: Quantities (multivalue)
- Field 36: Prices (multivalue)

---

## 🎯 Success Criteria

When everything is working correctly, you should see:

1. **Console Logs:**
   ```
   🔍 Loading drafts for user: EC
   📡 Fetching drafts from: GET_SO_TMP.XML?user_initials=EC
   📥 Response received - Status: 200
   ✅ Found drafts: 2
   ```

2. **UI Behavior:**
   - "My Drafts" modal opens instantly
   - Shows list of saved drafts with dates/times
   - Can click to load a draft
   - Can delete drafts

3. **Save Behavior:**
   - "Save for Later" shows success notification
   - Draft appears in "My Drafts" list
   - Can reload page and draft is still there

---

## 📞 Need Help?

If you're still having issues after following these steps:

1. **Check server logs** for any Pick/BASIC errors
2. **Verify database exists:** `:LIST-FILES` should show TMP
3. **Test with simple query:** `:SELECT TMP`
4. **Check permissions:** Ensure web user can read/write TMP

---

## 🔗 Related Files

- Frontend: `SO_EDIT.HTM` (already updated ✅)
- Backend: `GET_SO_TMP.XML`, `SAVE_SO_TMP.XML`, `DELETE_SO_TMP.XML`
- Documentation: This file

---

**Last Updated:** October 3, 2025
**Status:** Frontend ready ✅ | Backend needs deployment ⚠️
