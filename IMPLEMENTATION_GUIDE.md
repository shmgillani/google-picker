# Lumexa Extension Integration Guide

## 🎯 **Complete Implementation Solutions**

### **Solution 1: Iframe + localStorage (Recommended)**

#### **Step 1: Deploy HTML to Vercel**
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel --prod

# You'll get a URL like: https://your-app.vercel.app
```

#### **Step 2: Lumexa Extension Code**

```javascript
// In your Lumexa extension

class DriveFilePicker {
    constructor() {
        this.pickerUrl = 'https://your-app.vercel.app'; // Your deployed URL
        this.iframe = null;
    }

    // Method 1: Open in iframe
    openPickerInIframe(containerId) {
        // Store token in localStorage
        const tokenData = {
            access_token: 'your_access_token_here',
            expires_in: 3599,
            refresh_token: 'your_refresh_token_here',
            scope: 'https://www.googleapis.com/auth/drive.file'
        };
        
        localStorage.setItem('lumexa_drive_token', JSON.stringify(tokenData));

        // Create iframe
        this.iframe = document.createElement('iframe');
        this.iframe.src = this.pickerUrl;
        this.iframe.style.width = '100%';
        this.iframe.style.height = '600px';
        this.iframe.style.border = 'none';
        this.iframe.style.borderRadius = '8px';

        // Listen for messages from iframe
        window.addEventListener('message', this.handlePickerMessage.bind(this));

        // Add to container
        document.getElementById(containerId).appendChild(this.iframe);
    }

    // Method 2: Open in popup window
    openPickerInPopup() {
        // Store token
        const tokenData = {
            access_token: 'your_access_token_here',
            expires_in: 3599,
            refresh_token: 'your_refresh_token_here',
            scope: 'https://www.googleapis.com/auth/drive.file'
        };
        
        localStorage.setItem('lumexa_drive_token', JSON.stringify(tokenData));

        // Open popup
        const popup = window.open(
            this.pickerUrl,
            'drivePicker',
            'width=800,height=600,scrollbars=yes,resizable=yes'
        );

        // Listen for messages
        window.addEventListener('message', this.handlePickerMessage.bind(this));
    }

    handlePickerMessage(event) {
        if (event.origin !== 'https://your-app.vercel.app') return;

        switch (event.data.type) {
            case 'FILE_SELECTED':
                console.log('File selected:', event.data.data);
                this.onFileSelected(event.data.data);
                break;
            case 'CLOSE_PICKER':
                this.closePicker();
                break;
        }
    }

    onFileSelected(fileData) {
        // Handle selected file
        console.log('Selected file ID:', fileData.id);
        console.log('Selected file name:', fileData.name);
        
        // Send to your backend
        this.sendFileToBackend(fileData);
        
        // Close picker
        this.closePicker();
    }

    sendFileToBackend(fileData) {
        // Send to your Lumexa backend
        fetch('/api/process-file', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                fileId: fileData.id,
                fileName: fileData.name,
                fileType: fileData.mimeType,
                fileSize: fileData.sizeBytes,
                fileUrl: fileData.url
            })
        });
    }

    closePicker() {
        if (this.iframe) {
            this.iframe.remove();
            this.iframe = null;
        }
        // Remove event listener
        window.removeEventListener('message', this.handlePickerMessage.bind(this));
    }
}

// Usage in your extension
const picker = new DriveFilePicker();

// When user clicks "Select File" button
document.getElementById('selectFileBtn').addEventListener('click', () => {
    picker.openPickerInIframe('pickerContainer');
});
```

### **Solution 2: Popup Window + localStorage**

```javascript
// Alternative: Open in popup window
function openDrivePicker() {
    // Store token
    localStorage.setItem('lumexa_drive_token', JSON.stringify({
        access_token: 'your_token_here',
        expires_in: 3599,
        refresh_token: 'your_refresh_token_here',
        scope: 'https://www.googleapis.com/auth/drive.file'
    }));

    // Open popup
    const popup = window.open(
        'https://your-app.vercel.app',
        'drivePicker',
        'width=800,height=600'
    );

    // Check for file selection
    const checkClosed = setInterval(() => {
        if (popup.closed) {
            clearInterval(checkClosed);
            const selectedFile = localStorage.getItem('lumexa_selected_file');
            if (selectedFile) {
                const fileData = JSON.parse(selectedFile);
                console.log('File selected:', fileData);
                // Process file...
            }
        }
    }, 1000);
}
```

### **Solution 3: Direct Integration (No Hosting)**

```javascript
// If you want to embed directly in your extension
function createDrivePicker() {
    const pickerHTML = `
        <!DOCTYPE html>
        <html>
        <head>
            <title>Drive Picker</title>
            <script src="https://apis.google.com/js/api.js"></script>
            <script src="https://apis.google.com/js/picker.js"></script>
        </head>
        <body>
            <!-- Your picker HTML here -->
        </body>
        </html>
    `;
    
    // Create blob URL
    const blob = new Blob([pickerHTML], { type: 'text/html' });
    const url = URL.createObjectURL(blob);
    
    // Open in iframe
    const iframe = document.createElement('iframe');
    iframe.src = url;
    document.body.appendChild(iframe);
}
```

## 🔧 **Implementation Steps**

### **Step 1: Deploy HTML**
1. Deploy `index.html` to Vercel/Netlify
2. Get the public URL
3. Update the URL in your extension code

### **Step 2: Integrate in Lumexa Extension**
1. Add the `DriveFilePicker` class to your extension
2. Call `openPickerInIframe()` or `openPickerInPopup()` when user clicks "Select File"
3. Handle the `onFileSelected` callback to process the selected file

### **Step 3: Backend Integration**
1. Send selected file data to your Lumexa backend
2. Process the file using Google Drive API
3. Store file information in your database

## 📋 **Data Flow**

```
Lumexa Extension → Store Token → Open Picker → User Selects File → Store File Data → Close Picker → Send to Backend
```

## 🎯 **Key Features**

- ✅ **Token Management** - Automatically loads from localStorage
- ✅ **File Selection** - Full Google Drive picker functionality
- ✅ **Data Transfer** - Stores selected file in localStorage
- ✅ **Auto-close** - Closes picker after selection
- ✅ **Error Handling** - Proper error messages
- ✅ **Cross-origin** - Works with iframe/popup

## 🚀 **Quick Start**

1. **Deploy HTML** to Vercel
2. **Copy the DriveFilePicker class** to your extension
3. **Update the URL** in the class
4. **Call openPickerInIframe()** when user wants to select files
5. **Handle onFileSelected()** to process the selected file

This solution gives you a clean, integrated file picker that works seamlessly with your Lumexa extension!
