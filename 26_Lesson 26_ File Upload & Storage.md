# Lesson 26: File Upload & Storage

## 🎯 **Learning Objectives**
- Implement file upload functionality in React Native and Node.js
- Handle different file types (images, documents, videos)
- Set up cloud storage solutions (AWS S3, Cloudinary)
- Implement file validation and security measures
- Create efficient file processing and optimization

## 📚 **Table of Contents**
1. [File Upload Basics](#file-upload-basics)
2. [Client-Side File Handling](#client-side-file-handling)
3. [Server-Side File Processing](#server-side-file-processing)
4. [Cloud Storage Integration](#cloud-storage-integration)
5. [File Validation & Security](#file-validation--security)
6. [Image Processing & Optimization](#image-processing--optimization)
7. [Video Upload & Streaming](#video-upload--streaming)
8. [Document Management](#document-management)
9. [Progress Tracking](#progress-tracking)
10. [Practical Examples](#practical-examples)

---

## 📁 **File Upload Basics**

### **Understanding File Upload Process**
```
1. Client Selection → 2. Validation → 3. Upload → 4. Processing → 5. Storage → 6. Response
```

### **File Upload Methods**
- **Form Data**: Traditional multipart/form-data uploads
- **Base64**: Convert files to base64 strings
- **Binary**: Direct binary data transfer
- **Chunked**: Large file uploads in chunks

---

## 📱 **Client-Side File Handling**

### **React Native File Picker**
```javascript
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, Image, StyleSheet, Alert } from 'react-native';
import ImagePicker from 'react-native-image-picker';
import DocumentPicker from 'react-native-document-picker';
import { request, PERMISSIONS } from 'react-native-permissions';

const FileUploader = () => {
  const [selectedFile, setSelectedFile] = useState(null);
  const [uploadProgress, setUploadProgress] = useState(0);
  const [isUploading, setIsUploading] = useState(false);

  // Request permissions
  const requestPermissions = async () => {
    try {
      const cameraPermission = await request(PERMISSIONS.ANDROID.CAMERA);
      const storagePermission = await request(PERMISSIONS.ANDROID.WRITE_EXTERNAL_STORAGE);
      
      return cameraPermission === 'granted' && storagePermission === 'granted';
    } catch (error) {
      console.error('Permission error:', error);
      return false;
    }
  };

  // Select image from gallery
  const selectImage = () => {
    const options = {
      title: 'Select Image',
      storageOptions: {
        skipBackup: true,
        path: 'images',
      },
      quality: 0.8,
      maxWidth: 1024,
      maxHeight: 1024,
    };

    ImagePicker.showImagePicker(options, (response) => {
      if (response.didCancel) {
        console.log('User cancelled image picker');
      } else if (response.error) {
        console.error('ImagePicker Error: ', response.error);
        Alert.alert('Error', 'Failed to select image');
      } else {
        const file = {
          uri: response.uri,
          type: response.type || 'image/jpeg',
          name: response.fileName || `image_${Date.now()}.jpg`,
          size: response.fileSize,
        };
        setSelectedFile(file);
      }
    });
  };

  // Take photo with camera
  const takePhoto = () => {
    const options = {
      title: 'Take Photo',
      storageOptions: {
        skipBackup: true,
        path: 'images',
      },
      quality: 0.8,
      maxWidth: 1024,
      maxHeight: 1024,
    };

    ImagePicker.launchCamera(options, (response) => {
      if (response.didCancel) {
        console.log('User cancelled camera');
      } else if (response.error) {
        console.error('Camera Error: ', response.error);
        Alert.alert('Error', 'Failed to take photo');
      } else {
        const file = {
          uri: response.uri,
          type: response.type || 'image/jpeg',
          name: response.fileName || `photo_${Date.now()}.jpg`,
          size: response.fileSize,
        };
        setSelectedFile(file);
      }
    });
  };

  // Select document
  const selectDocument = async () => {
    try {
      const result = await DocumentPicker.pick({
        type: [DocumentPicker.types.allFiles],
      });

      const file = {
        uri: result.uri,
        type: result.type,
        name: result.name,
        size: result.size,
      };
      setSelectedFile(file);
    } catch (error) {
      if (!DocumentPicker.isCancel(error)) {
        console.error('Document picker error:', error);
        Alert.alert('Error', 'Failed to select document');
      }
    }
  };

  // Upload file to server
  const uploadFile = async () => {
    if (!selectedFile) {
      Alert.alert('Error', 'Please select a file first');
      return;
    }

    setIsUploading(true);
    setUploadProgress(0);

    try {
      const formData = new FormData();
      formData.append('file', {
        uri: selectedFile.uri,
        type: selectedFile.type,
        name: selectedFile.name,
      });

      const response = await fetch('http://localhost:3000/api/upload', {
        method: 'POST',
        body: formData,
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });

      if (!response.ok) {
        throw new Error('Upload failed');
      }

      const result = await response.json();
      Alert.alert('Success', 'File uploaded successfully!');
      setSelectedFile(null);
    } catch (error) {
      console.error('Upload error:', error);
      Alert.alert('Error', 'Failed to upload file');
    } finally {
      setIsUploading(false);
      setUploadProgress(0);
    }
  };

  // Format file size
  const formatFileSize = (bytes) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>File Uploader</Text>

      {/* File Preview */}
      {selectedFile && (
        <View style={styles.filePreview}>
          {selectedFile.type.startsWith('image/') ? (
            <Image source={{ uri: selectedFile.uri }} style={styles.imagePreview} />
          ) : (
            <View style={styles.documentPreview}>
              <Text style={styles.documentIcon}>📄</Text>
              <Text style={styles.documentName} numberOfLines={1}>
                {selectedFile.name}
              </Text>
            </View>
          )}
          <Text style={styles.fileInfo}>
            Size: {formatFileSize(selectedFile.size)}
          </Text>
        </View>
      )}

      {/* Action Buttons */}
      <View style={styles.buttonContainer}>
        <TouchableOpacity style={styles.button} onPress={selectImage}>
          <Text style={styles.buttonText}>Select Image</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.cameraButton]} onPress={takePhoto}>
          <Text style={styles.buttonText}>Take Photo</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.documentButton]} onPress={selectDocument}>
          <Text style={styles.buttonText}>Select Document</Text>
        </TouchableOpacity>
      </View>

      {/* Upload Button */}
      {selectedFile && (
        <TouchableOpacity
          style={[styles.uploadButton, isUploading && styles.disabledButton]}
          onPress={uploadFile}
          disabled={isUploading}
        >
          <Text style={styles.uploadButtonText}>
            {isUploading ? 'Uploading...' : 'Upload File'}
          </Text>
        </TouchableOpacity>
      )}

      {/* Progress Indicator */}
      {isUploading && (
        <View style={styles.progressContainer}>
          <View style={[styles.progressBar, { width: `${uploadProgress}%` }]} />
          <Text style={styles.progressText}>{uploadProgress}%</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  filePreview: {
    backgroundColor: 'white',
    borderRadius: 10,
    padding: 15,
    marginBottom: 20,
    alignItems: 'center',
  },
  imagePreview: {
    width: 200,
    height: 200,
    borderRadius: 10,
    marginBottom: 10,
  },
  documentPreview: {
    alignItems: 'center',
    marginBottom: 10,
  },
  documentIcon: {
    fontSize: 48,
    marginBottom: 10,
  },
  documentName: {
    fontSize: 16,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  fileInfo: {
    fontSize: 14,
    color: '#666',
  },
  buttonContainer: {
    gap: 10,
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  cameraButton: {
    backgroundColor: '#34C759',
  },
  documentButton: {
    backgroundColor: '#FF9500',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  uploadButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 20,
  },
  disabledButton: {
    backgroundColor: '#ccc',
  },
  uploadButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  progressContainer: {
    height: 20,
    backgroundColor: '#e0e0e0',
    borderRadius: 10,
    overflow: 'hidden',
    marginBottom: 10,
  },
  progressBar: {
    height: '100%',
    backgroundColor: '#007AFF',
  },
  progressText: {
    position: 'absolute',
    width: '100%',
    textAlign: 'center',
    lineHeight: 20,
    fontSize: 12,
    fontWeight: 'bold',
    color: '#333',
  },
});

export default FileUploader;
```

---

## 🖥️ **Server-Side File Processing**

### **Express File Upload Server**
```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;
const crypto = require('crypto');

const app = express();

// Create uploads directory if it doesn't exist
const uploadsDir = path.join(__dirname, 'uploads');
fs.mkdir(uploadsDir, { recursive: true }).catch(console.error);

// Configure multer for file uploads
const storage = multer.diskStorage({
  destination: async (req, file, cb) => {
    // Create user-specific directory
    const userId = req.user?.id || 'anonymous';
    const userDir = path.join(uploadsDir, userId);
    await fs.mkdir(userDir, { recursive: true });
    cb(null, userDir);
  },
  filename: (req, file, cb) => {
    // Generate unique filename
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const extension = path.extname(file.originalname);
    const basename = path.basename(file.originalname, extension);
    cb(null, `${basename}-${uniqueSuffix}${extension}`);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  // Allowed file types
  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'image/webp',
    'application/pdf',
    'text/plain',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  ];

  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error(`File type ${file.mimetype} is not allowed`), false);
  }
};

// Configure multer upload
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB limit
    files: 5, // Maximum 5 files
  },
});

// Error handling middleware for multer
const handleMulterError = (error, req, res, next) => {
  if (error instanceof multer.MulterError) {
    if (error.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({
        success: false,
        error: 'File too large. Maximum size is 10MB.',
      });
    }
    if (error.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({
        success: false,
        error: 'Too many files. Maximum 5 files allowed.',
      });
    }
  }

  if (error.message.includes('not allowed')) {
    return res.status(400).json({
      success: false,
      error: error.message,
    });
  }

  next(error);
};

// Single file upload
app.post('/api/upload/single', upload.single('file'), handleMulterError, async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        success: false,
        error: 'No file uploaded',
      });
    }

    // Save file metadata to database
    const fileData = {
      filename: req.file.filename,
      originalName: req.file.originalname,
      mimetype: req.file.mimetype,
      size: req.file.size,
      path: req.file.path,
      url: `/uploads/${req.user?.id || 'anonymous'}/${req.file.filename}`,
      uploadedBy: req.user?.id,
      uploadedAt: new Date(),
    };

    // Save to database (assuming you have a File model)
    // const file = await File.create(fileData);

    res.json({
      success: true,
      message: 'File uploaded successfully',
      file: fileData,
    });
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to upload file',
    });
  }
});

// Multiple files upload
app.post('/api/upload/multiple', upload.array('files', 5), handleMulterError, async (req, res) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({
        success: false,
        error: 'No files uploaded',
      });
    }

    const uploadedFiles = req.files.map(file => ({
      filename: file.filename,
      originalName: file.originalname,
      mimetype: file.mimetype,
      size: file.size,
      path: file.path,
      url: `/uploads/${req.user?.id || 'anonymous'}/${file.filename}`,
      uploadedBy: req.user?.id,
      uploadedAt: new Date(),
    }));

    // Save to database
    // const files = await File.insertMany(uploadedFiles);

    res.json({
      success: true,
      message: `${uploadedFiles.length} files uploaded successfully`,
      files: uploadedFiles,
    });
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to upload files',
    });
  }
});

// Mixed file upload (files + data)
app.post('/api/upload/mixed', upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'documents', maxCount: 3 },
  { name: 'images', maxCount: 5 },
]), handleMulterError, async (req, res) => {
  try {
    const files = {
      avatar: req.files.avatar?.[0],
      documents: req.files.documents || [],
      images: req.files.images || [],
    };

    // Additional form data
    const { title, description } = req.body;

    // Process and save files
    const processedFiles = [];

    Object.entries(files).forEach(([type, fileList]) => {
      if (Array.isArray(fileList)) {
        fileList.forEach(file => {
          processedFiles.push({
            type,
            filename: file.filename,
            originalName: file.originalname,
            mimetype: file.mimetype,
            size: file.size,
            path: file.path,
            url: `/uploads/${req.user?.id || 'anonymous'}/${file.filename}`,
          });
        });
      } else if (fileList) {
        processedFiles.push({
          type,
          filename: fileList.filename,
          originalName: fileList.originalname,
          mimetype: fileList.mimetype,
          size: fileList.size,
          path: fileList.path,
          url: `/uploads/${req.user?.id || 'anonymous'}/${fileList.filename}`,
        });
      }
    });

    res.json({
      success: true,
      message: 'Files uploaded successfully',
      data: {
        title,
        description,
        files: processedFiles,
      },
    });
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to upload files',
    });
  }
});

// Serve uploaded files
app.use('/uploads', express.static(uploadsDir));

// Get user's files
app.get('/api/files', async (req, res) => {
  try {
    const userId = req.user?.id;
    if (!userId) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required',
      });
    }

    // Get files from database
    // const files = await File.find({ uploadedBy: userId }).sort({ uploadedAt: -1 });

    // For now, return mock data
    const files = [
      {
        id: '1',
        filename: 'example.jpg',
        originalName: 'Example Image.jpg',
        mimetype: 'image/jpeg',
        size: 1024000,
        url: '/uploads/user1/example.jpg',
        uploadedAt: new Date(),
      },
    ];

    res.json({
      success: true,
      files,
    });
  } catch (error) {
    console.error('Get files error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to get files',
    });
  }
});

// Delete file
app.delete('/api/files/:filename', async (req, res) => {
  try {
    const userId = req.user?.id;
    const filename = req.params.filename;

    if (!userId) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required',
      });
    }

    // Check if file exists and belongs to user
    const filePath = path.join(uploadsDir, userId, filename);
    const fileExists = await fs.access(filePath).then(() => true).catch(() => false);

    if (!fileExists) {
      return res.status(404).json({
        success: false,
        error: 'File not found',
      });
    }

    // Delete file from filesystem
    await fs.unlink(filePath);

    // Delete from database
    // await File.findOneAndDelete({ filename, uploadedBy: userId });

    res.json({
      success: true,
      message: 'File deleted successfully',
    });
  } catch (error) {
    console.error('Delete file error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to delete file',
    });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`File upload server running on port ${PORT}`);
});
```

---

## ☁️ **Cloud Storage Integration**

### **AWS S3 Integration**
```javascript
const AWS = require('aws-sdk');
const multer = require('multer');
const multerS3 = require('multer-s3');

// Configure AWS
AWS.config.update({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: process.env.AWS_REGION || 'us-east-1',
});

const s3 = new AWS.S3();

// Configure multer for S3
const s3Storage = multerS3({
  s3: s3,
  bucket: process.env.AWS_S3_BUCKET,
  acl: 'public-read', // or 'private' for signed URLs
  metadata: (req, file, cb) => {
    cb(null, {
      fieldName: file.fieldname,
      originalName: file.originalname,
    });
  },
  key: (req, file, cb) => {
    // Generate unique key
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const extension = path.extname(file.originalname);
    const basename = path.basename(file.originalname, extension);
    const key = `uploads/${req.user?.id || 'anonymous'}/${basename}-${uniqueSuffix}${extension}`;
    cb(null, key);
  },
});

// File filter for S3
const s3FileFilter = (req, file, cb) => {
  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'image/webp',
    'application/pdf',
    'video/mp4',
    'video/avi',
  ];

  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error(`File type ${file.mimetype} is not allowed`), false);
  }
};

// Configure S3 upload
const s3Upload = multer({
  storage: s3Storage,
  fileFilter: s3FileFilter,
  limits: {
    fileSize: 50 * 1024 * 1024, // 50MB for videos
  },
});

// Upload to S3
app.post('/api/upload/s3', s3Upload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        success: false,
        error: 'No file uploaded',
      });
    }

    const fileData = {
      filename: req.file.key,
      originalName: req.file.originalname,
      mimetype: req.file.mimetype,
      size: req.file.size,
      url: req.file.location, // S3 URL
      bucket: req.file.bucket,
      etag: req.file.etag,
      uploadedBy: req.user?.id,
      uploadedAt: new Date(),
    };

    // Save metadata to database
    // const file = await File.create(fileData);

    res.json({
      success: true,
      message: 'File uploaded to S3 successfully',
      file: fileData,
    });
  } catch (error) {
    console.error('S3 upload error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to upload file to S3',
    });
  }
});

// Generate signed URL for private files
app.get('/api/files/:key/signed-url', async (req, res) => {
  try {
    const key = req.params.key;
    const userId = req.user?.id;

    // Verify file ownership
    // const file = await File.findOne({ filename: key, uploadedBy: userId });
    // if (!file) {
    //   return res.status(404).json({ error: 'File not found' });
    // }

    const signedUrl = s3.getSignedUrl('getObject', {
      Bucket: process.env.AWS_S3_BUCKET,
      Key: key,
      Expires: 3600, // 1 hour
    });

    res.json({
      success: true,
      signedUrl,
    });
  } catch (error) {
    console.error('Signed URL error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to generate signed URL',
    });
  }
});

// Delete file from S3
app.delete('/api/files/s3/:key', async (req, res) => {
  try {
    const key = req.params.key;
    const userId = req.user?.id;

    // Verify file ownership
    // const file = await File.findOne({ filename: key, uploadedBy: userId });
    // if (!file) {
    //   return res.status(404).json({ error: 'File not found' });
    // }

    await s3.deleteObject({
      Bucket: process.env.AWS_S3_BUCKET,
      Key: key,
    }).promise();

    // Remove from database
    // await File.findOneAndDelete({ filename: key, uploadedBy: userId });

    res.json({
      success: true,
      message: 'File deleted from S3 successfully',
    });
  } catch (error) {
    console.error('S3 delete error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to delete file from S3',
    });
  }
});
```

### **Cloudinary Integration**
```javascript
const cloudinary = require('cloudinary').v2;
const multer = require('multer');
const { CloudinaryStorage } = require('multer-storage-cloudinary');

// Configure Cloudinary
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

// Configure multer for Cloudinary
const cloudinaryStorage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: 'uploads',
    allowed_formats: ['jpg', 'png', 'gif', 'webp', 'pdf', 'mp4', 'avi'],
    transformation: [
      { width: 1000, height: 1000, crop: 'limit' }, // Resize images
    ],
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const extension = path.extname(file.originalname);
    const basename = path.basename(file.originalname, extension);
    cb(null, `${basename}-${uniqueSuffix}`);
  },
});

// Configure Cloudinary upload
const cloudinaryUpload = multer({
  storage: cloudinaryStorage,
  limits: {
    fileSize: 50 * 1024 * 1024, // 50MB
  },
});

// Upload to Cloudinary
app.post('/api/upload/cloudinary', cloudinaryUpload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        success: false,
        error: 'No file uploaded',
      });
    }

    const fileData = {
      filename: req.file.filename,
      originalName: req.file.originalname,
      mimetype: req.file.mimetype,
      size: req.file.size,
      url: req.file.path, // Cloudinary URL
      publicId: req.file.filename,
      cloudinaryId: req.file.filename,
      uploadedBy: req.user?.id,
      uploadedAt: new Date(),
    };

    // Save metadata to database
    // const file = await File.create(fileData);

    res.json({
      success: true,
      message: 'File uploaded to Cloudinary successfully',
      file: fileData,
    });
  } catch (error) {
    console.error('Cloudinary upload error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to upload file to Cloudinary',
    });
  }
});

// Transform image
app.get('/api/files/cloudinary/:publicId/transform', async (req, res) => {
  try {
    const { publicId } = req.params;
    const { width, height, quality, format } = req.query;

    const transformation = [];
    if (width) transformation.push({ width: parseInt(width) });
    if (height) transformation.push({ height: parseInt(height) });
    if (quality) transformation.push({ quality: quality });
    if (format) transformation.push({ format: format });

    const transformedUrl = cloudinary.url(publicId, {
      transformation,
    });

    res.json({
      success: true,
      transformedUrl,
    });
  } catch (error) {
    console.error('Transform error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to transform image',
    });
  }
});

// Delete from Cloudinary
app.delete('/api/files/cloudinary/:publicId', async (req, res) => {
  try {
    const { publicId } = req.params;
    const userId = req.user?.id;

    // Verify file ownership
    // const file = await File.findOne({ cloudinaryId: publicId, uploadedBy: userId });
    // if (!file) {
    //   return res.status(404).json({ error: 'File not found' });
    // }

    const result = await cloudinary.uploader.destroy(publicId);

    if (result.result === 'ok') {
      // Remove from database
      // await File.findOneAndDelete({ cloudinaryId: publicId, uploadedBy: userId });

      res.json({
        success: true,
        message: 'File deleted from Cloudinary successfully',
      });
    } else {
      res.status(500).json({
        success: false,
        error: 'Failed to delete file from Cloudinary',
      });
    }
  } catch (error) {
    console.error('Cloudinary delete error:', error);
    res.status(500).json({
      success: false,
      error: 'Failed to delete file from Cloudinary',
    });
  }
});
```

---

## 🔒 **File Validation & Security**

### **Comprehensive File Validation**
```javascript
const sharp = require('sharp');
const fs = require('fs').promises;
const path = require('path');

class FileValidator {
  constructor() {
    this.allowedTypes = {
      image: ['image/jpeg', 'image/png', 'image/gif', 'image/webp'],
      document: ['application/pdf', 'text/plain', 'application/msword'],
      video: ['video/mp4', 'video/avi', 'video/mov'],
      audio: ['audio/mpeg', 'audio/wav', 'audio/ogg'],
    };

    this.maxSizes = {
      image: 5 * 1024 * 1024, // 5MB
      document: 10 * 1024 * 1024, // 10MB
      video: 50 * 1024 * 1024, // 50MB
      audio: 20 * 1024 * 1024, // 20MB
    };
  }

  // Validate file type
  validateFileType(file, allowedTypes = null) {
    const types = allowedTypes || Object.values(this.allowedTypes).flat();
    
    if (!types.includes(file.mimetype)) {
      throw new Error(`File type ${file.mimetype} is not allowed`);
    }
  }

  // Validate file size
  validateFileSize(file, maxSize = null) {
    const limit = maxSize || this.getMaxSizeForType(file.mimetype);
    
    if (file.size > limit) {
      const limitMB = (limit / (1024 * 1024)).toFixed(1);
      throw new Error(`File size exceeds ${limitMB}MB limit`);
    }
  }

  // Get max size for file type
  getMaxSizeForType(mimetype) {
    for (const [category, types] of Object.entries(this.allowedTypes)) {
      if (types.includes(mimetype)) {
        return this.maxSizes[category];
      }
    }
    return 1024 * 1024; // 1MB default
  }

  // Validate image dimensions and corruption
  async validateImage(file) {
    try {
      const buffer = await fs.readFile(file.path);
      const metadata = await sharp(buffer).metadata();

      // Check for minimum dimensions
      if (metadata.width < 10 || metadata.height < 10) {
        throw new Error('Image dimensions too small');
      }

      // Check for maximum dimensions
      if (metadata.width > 5000 || metadata.height > 5000) {
        throw new Error('Image dimensions too large');
      }

      // Verify image is not corrupted
      await sharp(buffer).stats();

      return metadata;
    } catch (error) {
      throw new Error('Invalid or corrupted image file');
    }
  }

  // Scan for malware (basic check)
  async scanForMalware(file) {
    const buffer = await fs.readFile(file.path);
    const fileString = buffer.toString('utf8', 0, 100); // First 100 characters

    // Check for common malware signatures
    const malwareSignatures = [
      '<script', // XSS attempts
      'eval(', // Code injection
      'javascript:', // JavaScript URLs
      '<?php', // PHP code
    ];

    for (const signature of malwareSignatures) {
      if (fileString.toLowerCase().includes(signature.toLowerCase())) {
        throw new Error('File contains potentially malicious content');
      }
    }
  }

  // Generate secure filename
  generateSecureFilename(originalName) {
    const extension = path.extname(originalName).toLowerCase();
    const basename = path.basename(originalName, extension);
    
    // Remove potentially dangerous characters
    const safeBasename = basename.replace(/[^a-zA-Z0-9\-_]/g, '_');
    
    // Add timestamp and random string
    const timestamp = Date.now();
    const random = Math.random().toString(36).substring(2, 8);
    
    return `${safeBasename}_${timestamp}_${random}${extension}`;
  }

  // Comprehensive file validation
  async validateFile(file, options = {}) {
    const {
      allowedTypes = null,
      maxSize = null,
      requireImageValidation = true,
      requireMalwareScan = true,
    } = options;

    // Basic validations
    this.validateFileType(file, allowedTypes);
    this.validateFileSize(file, maxSize);

    // Image-specific validations
    if (requireImageValidation && file.mimetype.startsWith('image/')) {
      await this.validateImage(file);
    }

    // Malware scan
    if (requireMalwareScan) {
      await this.scanForMalware(file);
    }

    // Generate secure filename
    const secureFilename = this.generateSecureFilename(file.originalname);

    return {
      isValid: true,
      secureFilename,
      metadata: file.mimetype.startsWith('image/') ? await this.validateImage(file) : null,
    };
  }
}

const fileValidator = new FileValidator();
module.exports = fileValidator;
```

---

## 🖼️ **Image Processing & Optimization**

### **Image Processing with Sharp**
```javascript
const sharp = require('sharp');
const fs = require('fs').promises;
const path = require('path');

class ImageProcessor {
  constructor() {
    this.quality = 80;
    this.maxWidth = 1920;
    this.maxHeight = 1080;
  }

  // Resize image
  async resizeImage(inputPath, outputPath, options = {}) {
    const { width, height, fit = 'cover' } = options;

    await sharp(inputPath)
      .resize(width, height, {
        fit,
        withoutEnlargement: true,
      })
      .jpeg({ quality: this.quality })
      .toFile(outputPath);

    return outputPath;
  }

  // Create thumbnail
  async createThumbnail(inputPath, outputPath, size = 300) {
    await sharp(inputPath)
      .resize(size, size, {
        fit: 'cover',
        position: 'center',
      })
      .jpeg({ quality: 70 })
      .toFile(outputPath);

    return outputPath;
  }

  // Optimize image
  async optimizeImage(inputPath, outputPath) {
    const metadata = await sharp(inputPath).metadata();

    let pipeline = sharp(inputPath);

    // Resize if too large
    if (metadata.width > this.maxWidth || metadata.height > this.maxHeight) {
      pipeline = pipeline.resize(this.maxWidth, this.maxHeight, {
        fit: 'inside',
        withoutEnlargement: true,
      });
    }

    // Convert to JPEG for web optimization
    if (metadata.format !== 'jpeg') {
      pipeline = pipeline.jpeg({
        quality: this.quality,
        progressive: true,
      });
    } else {
      pipeline = pipeline.jpeg({
        quality: this.quality,
        progressive: true,
      });
    }

    await pipeline.toFile(outputPath);
    return outputPath;
  }

  // Generate multiple sizes
  async generateSizes(inputPath, outputDir, filename) {
    const baseName = path.basename(filename, path.extname(filename));
    const sizes = [
      { suffix: '_thumb', size: 150 },
      { suffix: '_small', size: 400 },
      { suffix: '_medium', size: 800 },
      { suffix: '_large', size: 1200 },
    ];

    const results = {};

    for (const { suffix, size } of sizes) {
      const outputPath = path.join(outputDir, `${baseName}${suffix}.jpg`);
      
      await sharp(inputPath)
        .resize(size, size, {
          fit: 'inside',
          withoutEnlargement: true,
        })
        .jpeg({ quality: this.quality })
        .toFile(outputPath);

      results[suffix] = outputPath;
    }

    return results;
  }

  // Add watermark
  async addWatermark(inputPath, outputPath, watermarkPath) {
    const [inputMetadata, watermarkMetadata] = await Promise.all([
      sharp(inputPath).metadata(),
      sharp(watermarkPath).metadata(),
    ]);

    const watermarkWidth = Math.floor(inputMetadata.width * 0.2);
    const watermarkHeight = Math.floor(watermarkWidth * (watermarkMetadata.height / watermarkMetadata.width));

    await sharp(inputPath)
      .composite([{
        input: await sharp(watermarkPath)
          .resize(watermarkWidth, watermarkHeight)
          .png()
          .toBuffer(),
        top: inputMetadata.height - watermarkHeight - 20,
        left: inputMetadata.width - watermarkWidth - 20,
        opacity: 0.7,
      }])
      .jpeg({ quality: this.quality })
      .toFile(outputPath);

    return outputPath;
  }

  // Convert format
  async convertFormat(inputPath, outputPath, format = 'jpeg') {
    let pipeline = sharp(inputPath);

    switch (format) {
      case 'jpeg':
      case 'jpg':
        pipeline = pipeline.jpeg({ quality: this.quality });
        break;
      case 'png':
        pipeline = pipeline.png({ compressionLevel: 6 });
        break;
      case 'webp':
        pipeline = pipeline.webp({ quality: this.quality });
        break;
      default:
        throw new Error(`Unsupported format: ${format}`);
    }

    await pipeline.toFile(outputPath);
    return outputPath;
  }

  // Get image metadata
  async getMetadata(inputPath) {
    const metadata = await sharp(inputPath).metadata();
    const stats = await sharp(inputPath).stats();

    return {
      ...metadata,
      dominantColor: stats.dominant,
      channels: stats.channels,
      isOpaque: stats.isOpaque,
      entropy: stats.entropy,
    };
  }

  // Batch process images
  async batchProcess(inputDir, outputDir, operations = []) {
    const files = await fs.readdir(inputDir);
    const imageFiles = files.filter(file => 
      /\.(jpg|jpeg|png|gif|webp)$/i.test(file)
    );

    const results = [];

    for (const file of imageFiles) {
      const inputPath = path.join(inputDir, file);
      const outputPath = path.join(outputDir, file);

      try {
        let processedPath = inputPath;

        for (const operation of operations) {
          const tempOutput = path.join(outputDir, `temp_${file}`);
          
          switch (operation.type) {
            case 'resize':
              processedPath = await this.resizeImage(processedPath, tempOutput, operation.options);
              break;
            case 'optimize':
              processedPath = await this.optimizeImage(processedPath, tempOutput);
              break;
            case 'thumbnail':
              processedPath = await this.createThumbnail(processedPath, tempOutput, operation.size);
              break;
            case 'convert':
              processedPath = await this.convertFormat(processedPath, tempOutput, operation.format);
              break;
          }

          // Move temp file to final output
          if (processedPath !== tempOutput) {
            await fs.rename(processedPath, tempOutput);
            processedPath = tempOutput;
          }
        }

        // Move to final output
        await fs.rename(processedPath, outputPath);

        results.push({
          file,
          success: true,
          outputPath,
        });
      } catch (error) {
        results.push({
          file,
          success: false,
          error: error.message,
        });
      }
    }

    return results;
  }
}

const imageProcessor = new ImageProcessor();
module.exports = imageProcessor;
```

---

## 🎥 **Video Upload & Streaming**

### **Video Upload with Processing**
```javascript
const ffmpeg = require('fluent-ffmpeg');
const fs = require('fs').promises;
const path = require('path');

class VideoProcessor {
  constructor() {
    this.ffmpegPath = process.env.FFMPEG_PATH || 'ffmpeg';
    ffmpeg.setFfmpegPath(this.ffmpegPath);
  }

  // Get video metadata
  async getVideoMetadata(inputPath) {
    return new Promise((resolve, reject) => {
      ffmpeg.ffprobe(inputPath, (err, metadata) => {
        if (err) reject(err);
        else resolve(metadata);
      });
    });
  }

  // Compress video
  async compressVideo(inputPath, outputPath, options = {}) {
    const {
      videoBitrate = '1000k',
      audioBitrate = '128k',
      resolution = '1280x720',
      preset = 'medium',
    } = options;

    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .videoCodec('libx264')
        .audioCodec('aac')
        .videoBitrate(videoBitrate)
        .audioBitrate(audioBitrate)
        .size(resolution)
        .preset(preset)
        .on('end', () => resolve(outputPath))
        .on('error', reject)
        .save(outputPath);
    });
  }

  // Generate thumbnail from video
  async generateThumbnail(inputPath, outputPath, timestamp = '00:00:01') {
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .screenshots({
          timestamps: [timestamp],
          filename: path.basename(outputPath),
          folder: path.dirname(outputPath),
          size: '320x240',
        })
        .on('end', () => resolve(outputPath))
        .on('error', reject);
    });
  }

  // Convert video format
  async convertVideoFormat(inputPath, outputPath, format = 'mp4') {
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .toFormat(format)
        .on('end', () => resolve(outputPath))
        .on('error', reject)
        .save(outputPath);
    });
  }

  // Extract audio from video
  async extractAudio(inputPath, outputPath) {
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .noVideo()
        .audioCodec('libmp3lame')
        .audioBitrate('192k')
        .on('end', () => resolve(outputPath))
        .on('error', reject)
        .save(outputPath);
    });
  }

  // Create video preview (short clip)
  async createPreview(inputPath, outputPath, duration = 30) {
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .duration(duration)
        .on('end', () => resolve(outputPath))
        .on('error', reject)
        .save(outputPath);
    });
  }

  // Get video quality info
  async analyzeVideoQuality(inputPath) {
    const metadata = await this.getVideoMetadata(inputPath);
    const videoStream = metadata.streams.find(s => s.codec_type === 'video');

    if (!videoStream) {
      throw new Error('No video stream found');
    }

    return {
      duration: metadata.format.duration,
      size: metadata.format.size,
      bitrate: metadata.format.bit_rate,
      width: videoStream.width,
      height: videoStream.height,
      codec: videoStream.codec_name,
      fps: videoStream.r_frame_rate,
      aspectRatio: videoStream.width / videoStream.height,
    };
  }

  // Batch process videos
  async batchProcessVideos(inputDir, outputDir, operations = []) {
    const files = await fs.readdir(inputDir);
    const videoFiles = files.filter(file => 
      /\.(mp4|avi|mov|mkv|wmv|flv|webm)$/i.test(file)
    );

    const results = [];

    for (const file of videoFiles) {
      const inputPath = path.join(inputDir, file);
      const baseName = path.basename(file, path.extname(file));

      try {
        const result = { file, operations: {} };

        for (const operation of operations) {
          const outputFileName = `${baseName}_${operation.type}${path.extname(file)}`;
          const outputPath = path.join(outputDir, outputFileName);

          switch (operation.type) {
            case 'compress':
              await this.compressVideo(inputPath, outputPath, operation.options);
              result.operations.compress = outputPath;
              break;
            case 'thumbnail':
              const thumbOutput = path.join(outputDir, `${baseName}_thumbnail.jpg`);
              await this.generateThumbnail(inputPath, thumbOutput, operation.timestamp);
              result.operations.thumbnail = thumbOutput;
              break;
            case 'convert':
              const convertOutput = path.join(outputDir, `${baseName}_converted.${operation.format || 'mp4'}`);
              await this.convertVideoFormat(inputPath, convertOutput, operation.format);
              result.operations.convert = convertOutput;
              break;
            case 'extract_audio':
              const audioOutput = path.join(outputDir, `${baseName}_audio.mp3`);
              await this.extractAudio(inputPath, audioOutput);
              result.operations.extract_audio = audioOutput;
              break;
            case 'preview':
              const previewOutput = path.join(outputDir, `${baseName}_preview.mp4`);
              await this.createPreview(inputPath, previewOutput, operation.duration);
              result.operations.preview = previewOutput;
              break;
            default:
              throw new Error(`Unknown operation type: ${operation.type}`);
          }
        }

        results.push({
          ...result,
          success: true,
        });
      } catch (error) {
        results.push({
          file,
          success: false,
          error: error.message,
        });
      }
    }

    return results;
  }

  // Stream video with range support
  async streamVideo(req, res, videoPath) {
    const stat = await fs.stat(videoPath);
    const fileSize = stat.size;
    const range = req.headers.range;

    if (range) {
      const parts = range.replace(/bytes=/, '').split('-');
      const start = parseInt(parts[0], 10);
      const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
      const chunksize = (end - start) + 1;
      const file = await fs.open(videoPath, 'r');
      const buffer = Buffer.alloc(chunksize);

      await file.read(buffer, 0, chunksize, start);
      await file.close();

      res.writeHead(206, {
        'Content-Range': `bytes ${start}-${end}/${fileSize}`,
        'Accept-Ranges': 'bytes',
        'Content-Length': chunksize,
        'Content-Type': 'video/mp4',
      });

      res.end(buffer);
    } else {
      res.writeHead(200, {
        'Content-Length': fileSize,
        'Content-Type': 'video/mp4',
      });

      const file = await fs.createReadStream(videoPath);
      file.pipe(res);
    }
  }

  // Get video streaming URL
  getStreamingUrl(videoPath, baseUrl = '') {
    return `${baseUrl}/api/videos/stream/${encodeURIComponent(path.basename(videoPath))}`;
  }
}

const videoProcessor = new VideoProcessor();
module.exports = videoProcessor;
---

**Next Lesson**: [Lesson 27: Testing & Quality Assurance](Lesson%2027_%20Testing%20&%20Quality%20Assurance.md)