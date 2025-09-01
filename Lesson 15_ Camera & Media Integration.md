# Lesson 15: Camera & Media Integration

## 🎯 **Learning Objectives**
- Implement camera functionality in React Native apps
- Handle photo and video capture
- Integrate media libraries and galleries
- Process and manipulate captured media
- Implement real-time camera features

## 📚 **Table of Contents**
1. [Introduction to Camera Integration](#introduction-to-camera-integration)
2. [Camera Permissions](#camera-permissions)
3. [Basic Photo Capture](#basic-photo-capture)
4. [Video Recording](#video-recording)
5. [Media Library Integration](#media-library-integration)
6. [Real-time Camera Features](#real-time-camera-features)
7. [Image Processing](#image-processing)
8. [Advanced Camera Features](#advanced-camera-features)
9. [Practical Examples](#practical-examples)

---

## 📷 **Introduction to Camera Integration**

Camera integration is essential for modern mobile applications. React Native provides several libraries for camera access and media handling.

### **Popular Camera Libraries**
- **react-native-image-picker**: Simple photo/video selection
- **react-native-image-crop-picker**: Advanced image cropping and selection
- **react-native-camera**: Full camera control with real-time features
- **expo-camera**: Camera module for Expo projects

### **Use Cases**
- **Social Media Apps**: Photo/video sharing
- **E-commerce**: Product photography
- **Document Scanning**: OCR and document capture
- **Identity Verification**: Selfie and document capture
- **AR Applications**: Camera overlay features

---

## 🔐 **Camera Permissions**

### **Requesting Permissions**
```javascript
import { PermissionsAndroid, Platform } from 'react-native';

const requestCameraPermission = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.CAMERA,
        {
          title: 'Camera Permission',
          message: 'App needs camera access to take photos',
          buttonNeutral: 'Ask Me Later',
          buttonNegative: 'Cancel',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.warn(err);
      return false;
    }
  } else {
    // iOS permissions are handled automatically
    return true;
  }
};
```

### **Storage Permissions (Android)**
```javascript
const requestStoragePermission = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.WRITE_EXTERNAL_STORAGE,
        {
          title: 'Storage Permission',
          message: 'App needs storage access to save photos',
          buttonNeutral: 'Ask Me Later',
          buttonNegative: 'Cancel',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.warn(err);
      return false;
    }
  }
  return true;
};
```

---

## 📸 **Basic Photo Capture**

### **Using react-native-image-picker**
```javascript
import ImagePicker from 'react-native-image-picker';

const selectPhotoFromGallery = () => {
  const options = {
    title: 'Select Photo',
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
      console.log('ImagePicker Error: ', response.error);
    } else {
      // Handle the selected image
      const source = { uri: response.uri };
      console.log('Selected image:', source);
      // You can now use this image in your app
    }
  });
};
```

### **Using react-native-image-crop-picker**
```javascript
import ImagePicker from 'react-native-image-crop-picker';

const openCamera = () => {
  ImagePicker.openCamera({
    width: 300,
    height: 400,
    cropping: true,
    includeBase64: true,
    includeExif: true,
  }).then(image => {
    console.log('Captured image:', image);
    // image.path contains the path to the image
    // image.data contains base64 encoded image
  }).catch(error => {
    console.log('Camera error:', error);
  });
};

const selectMultipleImages = () => {
  ImagePicker.openPicker({
    multiple: true,
    maxFiles: 5,
    mediaType: 'photo',
    includeBase64: false,
  }).then(images => {
    console.log('Selected images:', images);
  }).catch(error => {
    console.log('Picker error:', error);
  });
};
```

---

## 🎥 **Video Recording**

### **Video Capture with Image Picker**
```javascript
const recordVideo = () => {
  const options = {
    title: 'Record Video',
    mediaType: 'video',
    videoQuality: 'high',
    durationLimit: 30, // 30 seconds
    storageOptions: {
      skipBackup: true,
      path: 'videos',
    },
  };

  ImagePicker.showImagePicker(options, (response) => {
    if (response.didCancel) {
      console.log('User cancelled video recording');
    } else if (response.error) {
      console.log('Video recording error:', response.error);
    } else {
      console.log('Video recorded:', response.uri);
      // response.uri contains the video file path
    }
  });
};
```

### **Advanced Video Recording**
```javascript
import ImagePicker from 'react-native-image-crop-picker';

const recordAdvancedVideo = () => {
  ImagePicker.openCamera({
    mediaType: 'video',
    width: 1920,
    height: 1080,
    videoQuality: 'high',
    durationLimit: 60,
    includeExif: true,
  }).then(video => {
    console.log('Video recorded:', video);
    // video.path contains the video file path
    // video.size contains file size
    // video.duration contains video duration
  }).catch(error => {
    console.log('Video recording error:', error);
  });
};
```

---

## 🖼️ **Media Library Integration**

### **Accessing Device Media**
```javascript
import { CameraRoll } from 'react-native';

const getPhotosFromLibrary = async () => {
  try {
    const photos = await CameraRoll.getPhotos({
      first: 20,
      assetType: 'Photos',
      include: ['filename', 'fileSize', 'imageSize', 'playableDuration'],
    });

    console.log('Photos:', photos.edges);
    return photos.edges;
  } catch (error) {
    console.error('Error getting photos:', error);
    return [];
  }
};
```

### **Creating Custom Media Picker**
```javascript
import React, { useState, useEffect } from 'react';
import { View, FlatList, Image, TouchableOpacity, StyleSheet, Dimensions } from 'react-native';
import { CameraRoll } from 'react-native';

const { width } = Dimensions.get('window');
const numColumns = 3;
const imageSize = width / numColumns;

const MediaPicker = ({ onSelectMedia, maxSelection = 5 }) => {
  const [photos, setPhotos] = useState([]);
  const [selectedPhotos, setSelectedPhotos] = useState([]);

  useEffect(() => {
    loadPhotos();
  }, []);

  const loadPhotos = async () => {
    try {
      const result = await CameraRoll.getPhotos({
        first: 100,
        assetType: 'All',
        include: ['filename', 'fileSize', 'imageSize', 'playableDuration'],
      });
      setPhotos(result.edges);
    } catch (error) {
      console.error('Error loading photos:', error);
    }
  };

  const togglePhotoSelection = (photo) => {
    const isSelected = selectedPhotos.find(item => item.node.image.uri === photo.node.image.uri);

    if (isSelected) {
      setSelectedPhotos(selectedPhotos.filter(item => item.node.image.uri !== photo.node.image.uri));
    } else if (selectedPhotos.length < maxSelection) {
      setSelectedPhotos([...selectedPhotos, photo]);
    }
  };

  const renderPhoto = ({ item }) => {
    const isSelected = selectedPhotos.find(photo => photo.node.image.uri === item.node.image.uri);

    return (
      <TouchableOpacity
        style={styles.photoContainer}
        onPress={() => togglePhotoSelection(item)}
      >
        <Image
          source={{ uri: item.node.image.uri }}
          style={[
            styles.photo,
            isSelected && styles.selectedPhoto
          ]}
        />
        {isSelected && (
          <View style={styles.selectionIndicator}>
            <Text style={styles.selectionText}>{selectedPhotos.indexOf(isSelected) + 1}</Text>
          </View>
        )}
      </TouchableOpacity>
    );
  };

  const handleDone = () => {
    onSelectMedia(selectedPhotos);
  };

  return (
    <View style={styles.container}>
      <FlatList
        data={photos}
        renderItem={renderPhoto}
        keyExtractor={(item) => item.node.image.uri}
        numColumns={numColumns}
      />
      {selectedPhotos.length > 0 && (
        <TouchableOpacity style={styles.doneButton} onPress={handleDone}>
          <Text style={styles.doneText}>Done ({selectedPhotos.length})</Text>
        </TouchableOpacity>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
  },
  photoContainer: {
    width: imageSize,
    height: imageSize,
    margin: 1,
  },
  photo: {
    width: '100%',
    height: '100%',
  },
  selectedPhoto: {
    opacity: 0.7,
  },
  selectionIndicator: {
    position: 'absolute',
    top: 5,
    right: 5,
    backgroundColor: '#007AFF',
    borderRadius: 12,
    width: 24,
    height: 24,
    justifyContent: 'center',
    alignItems: 'center',
  },
  selectionText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  doneButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    alignItems: 'center',
  },
  doneText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default MediaPicker;
```

---

## 📹 **Real-time Camera Features**

### **Using react-native-camera**
```javascript
import React, { useRef, useState } from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import { RNCamera } from 'react-native-camera';

const CameraScreen = () => {
  const cameraRef = useRef(null);
  const [isRecording, setIsRecording] = useState(false);

  const takePicture = async () => {
    if (cameraRef.current) {
      const options = {
        quality: 0.8,
        base64: false,
        width: 1024,
      };

      try {
        const data = await cameraRef.current.takePictureAsync(options);
        console.log('Picture taken:', data.uri);
      } catch (error) {
        console.error('Picture error:', error);
      }
    }
  };

  const startRecording = async () => {
    if (cameraRef.current && !isRecording) {
      setIsRecording(true);
      try {
        const data = await cameraRef.current.recordAsync({
          quality: RNCamera.Constants.VideoQuality['720p'],
          maxDuration: 30,
        });
        console.log('Video recorded:', data.uri);
      } catch (error) {
        console.error('Recording error:', error);
      } finally {
        setIsRecording(false);
      }
    }
  };

  const stopRecording = () => {
    if (cameraRef.current && isRecording) {
      cameraRef.current.stopRecording();
      setIsRecording(false);
    }
  };

  return (
    <View style={styles.container}>
      <RNCamera
        ref={cameraRef}
        style={styles.camera}
        type={RNCamera.Constants.Type.back}
        flashMode={RNCamera.Constants.FlashMode.off}
        captureAudio={true}
      />

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={takePicture}>
          <Text style={styles.buttonText}>Take Photo</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.button, isRecording && styles.recordingButton]}
          onPress={isRecording ? stopRecording : startRecording}
        >
          <Text style={styles.buttonText}>
            {isRecording ? 'Stop Recording' : 'Start Recording'}
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  camera: {
    flex: 1,
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    padding: 20,
    backgroundColor: 'rgba(0,0,0,0.5)',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    minWidth: 120,
    alignItems: 'center',
  },
  recordingButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default CameraScreen;
```

### **QR Code Scanner**
```javascript
import React, { useRef } from 'react';
import { View, Text, StyleSheet, Alert } from 'react-native';
import { RNCamera } from 'react-native-camera';

const QRScanner = () => {
  const cameraRef = useRef(null);

  const onBarCodeRead = (scanResult) => {
    console.log('QR Code detected:', scanResult.data);
    Alert.alert(
      'QR Code Detected',
      `Data: ${scanResult.data}`,
      [{ text: 'OK' }]
    );
  };

  return (
    <View style={styles.container}>
      <RNCamera
        ref={cameraRef}
        style={styles.camera}
        type={RNCamera.Constants.Type.back}
        flashMode={RNCamera.Constants.FlashMode.off}
        onBarCodeRead={onBarCodeRead}
        barCodeTypes={[RNCamera.Constants.BarCodeTypes.qr]}
      />

      <View style={styles.overlay}>
        <Text style={styles.instruction}>
          Point camera at QR code to scan
        </Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  camera: {
    flex: 1,
  },
  overlay: {
    position: 'absolute',
    bottom: 50,
    left: 0,
    right: 0,
    alignItems: 'center',
  },
  instruction: {
    color: 'white',
    fontSize: 16,
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 10,
    borderRadius: 5,
  },
});

export default QRScanner;
```

---

## 🖼️ **Image Processing**

### **Image Resizing and Compression**
```javascript
import ImageResizer from 'react-native-image-resizer';

const resizeImage = async (imageUri) => {
  try {
    const resizedImage = await ImageResizer.createResizedImage(
      imageUri,
      800, // new width
      600, // new height
      'JPEG', // format
      80, // quality (0-100)
      0, // rotation
      undefined, // output path (optional)
      false, // keep metadata
      { mode: 'contain' } // resize mode
    );

    console.log('Resized image:', resizedImage);
    return resizedImage;
  } catch (error) {
    console.error('Resize error:', error);
    return null;
  }
};
```

### **Image Manipulation**
```javascript
import { manipulateAsync, SaveFormat } from 'expo-image-manipulator';

const processImage = async (imageUri) => {
  try {
    const manipulatedImage = await manipulateAsync(
      imageUri,
      [
        { resize: { width: 800, height: 600 } },
        { rotate: 90 },
        { flip: 'horizontal' },
      ],
      {
        compress: 0.8,
        format: SaveFormat.JPEG,
      }
    );

    console.log('Processed image:', manipulatedImage);
    return manipulatedImage;
  } catch (error) {
    console.error('Processing error:', error);
    return null;
  }
};
```

---

## 🚀 **Advanced Camera Features**

### **Custom Camera Controls**
```javascript
import React, { useRef, useState } from 'react';
import { View, TouchableOpacity, Text, StyleSheet, Slider } from 'react-native';
import { RNCamera } from 'react-native-camera';

const AdvancedCamera = () => {
  const cameraRef = useRef(null);
  const [flashMode, setFlashMode] = useState(RNCamera.Constants.FlashMode.off);
  const [cameraType, setCameraType] = useState(RNCamera.Constants.Type.back);
  const [zoom, setZoom] = useState(0);
  const [isRecording, setIsRecording] = useState(false);

  const toggleFlash = () => {
    setFlashMode(
      flashMode === RNCamera.Constants.FlashMode.off
        ? RNCamera.Constants.FlashMode.on
        : RNCamera.Constants.FlashMode.off
    );
  };

  const toggleCamera = () => {
    setCameraType(
      cameraType === RNCamera.Constants.Type.back
        ? RNCamera.Constants.Type.front
        : RNCamera.Constants.Type.back
    );
  };

  const takePicture = async () => {
    if (cameraRef.current) {
      const options = {
        quality: 0.8,
        base64: false,
        width: 1024,
        flashMode: flashMode,
      };

      try {
        const data = await cameraRef.current.takePictureAsync(options);
        console.log('Picture taken:', data.uri);
      } catch (error) {
        console.error('Picture error:', error);
      }
    }
  };

  return (
    <View style={styles.container}>
      <RNCamera
        ref={cameraRef}
        style={styles.camera}
        type={cameraType}
        flashMode={flashMode}
        zoom={zoom}
        captureAudio={true}
      />

      {/* Zoom Control */}
      <View style={styles.zoomContainer}>
        <Text style={styles.zoomText}>Zoom: {Math.round(zoom * 100)}%</Text>
        <Slider
          style={styles.zoomSlider}
          minimumValue={0}
          maximumValue={1}
          value={zoom}
          onValueChange={setZoom}
          minimumTrackTintColor="#007AFF"
          maximumTrackTintColor="#ddd"
        />
      </View>

      {/* Camera Controls */}
      <View style={styles.controls}>
        <TouchableOpacity style={styles.controlButton} onPress={toggleFlash}>
          <Text style={styles.controlText}>
            Flash: {flashMode === RNCamera.Constants.FlashMode.on ? 'ON' : 'OFF'}
          </Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.captureButton} onPress={takePicture}>
          <Text style={styles.captureText}>📸</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.controlButton} onPress={toggleCamera}>
          <Text style={styles.controlText}>Flip</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  camera: {
    flex: 1,
  },
  zoomContainer: {
    position: 'absolute',
    top: 50,
    left: 20,
    right: 20,
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 10,
    borderRadius: 10,
  },
  zoomText: {
    color: 'white',
    textAlign: 'center',
    marginBottom: 5,
  },
  zoomSlider: {
    width: '100%',
  },
  controls: {
    position: 'absolute',
    bottom: 30,
    left: 0,
    right: 0,
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
  },
  controlButton: {
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 10,
    borderRadius: 5,
  },
  controlText: {
    color: 'white',
    fontSize: 12,
  },
  captureButton: {
    width: 70,
    height: 70,
    borderRadius: 35,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
  },
  captureText: {
    fontSize: 24,
  },
});

export default AdvancedCamera;
```

---

## 🎯 **Practical Examples**

### **Social Media Photo App**
```javascript
import React, { useState } from 'react';
import { View, Image, TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
import ImagePicker from 'react-native-image-crop-picker';

const SocialPhotoApp = () => {
  const [selectedImage, setSelectedImage] = useState(null);

  const selectImage = () => {
    ImagePicker.openPicker({
      width: 300,
      height: 300,
      cropping: true,
      includeBase64: true,
    }).then(image => {
      setSelectedImage(image);
    }).catch(error => {
      console.log('Image selection error:', error);
    });
  };

  const takePhoto = () => {
    ImagePicker.openCamera({
      width: 300,
      height: 300,
      cropping: true,
      includeBase64: true,
    }).then(image => {
      setSelectedImage(image);
    }).catch(error => {
      console.log('Camera error:', error);
    });
  };

  const uploadImage = () => {
    if (!selectedImage) {
      Alert.alert('Error', 'Please select an image first');
      return;
    }

    // Simulate upload
    Alert.alert('Success', 'Image uploaded successfully!');
  };

  return (
    <View style={styles.container}>
      <View style={styles.imageContainer}>
        {selectedImage ? (
          <Image source={{ uri: selectedImage.path }} style={styles.image} />
        ) : (
          <View style={styles.placeholder}>
            <Text style={styles.placeholderText}>No image selected</Text>
          </View>
        )}
      </View>

      <View style={styles.buttonContainer}>
        <TouchableOpacity style={styles.button} onPress={selectImage}>
          <Text style={styles.buttonText}>Select from Gallery</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.cameraButton]} onPress={takePhoto}>
          <Text style={styles.buttonText}>Take Photo</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.button, styles.uploadButton]}
          onPress={uploadImage}
          disabled={!selectedImage}
        >
          <Text style={[styles.buttonText, !selectedImage && styles.disabledText]}>
            Upload Image
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  imageContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 20,
  },
  image: {
    width: 300,
    height: 300,
    borderRadius: 10,
  },
  placeholder: {
    width: 300,
    height: 300,
    borderRadius: 10,
    backgroundColor: '#ddd',
    justifyContent: 'center',
    alignItems: 'center',
  },
  placeholderText: {
    color: '#666',
    fontSize: 16,
  },
  buttonContainer: {
    gap: 10,
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
  uploadButton: {
    backgroundColor: '#FF9500',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  disabledText: {
    opacity: 0.5,
  },
});

export default SocialPhotoApp;
```

### **Document Scanner**
```javascript
import React, { useRef, useState } from 'react';
import { View, TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
import { RNCamera } from 'react-native-camera';
import ImagePicker from 'react-native-image-crop-picker';

const DocumentScanner = () => {
  const cameraRef = useRef(null);
  const [scannedDocument, setScannedDocument] = useState(null);

  const scanDocument = async () => {
    if (cameraRef.current) {
      const options = {
        quality: 0.8,
        base64: false,
        width: 1024,
      };

      try {
        const data = await cameraRef.current.takePictureAsync(options);
        // Process the captured image for document scanning
        processDocument(data.uri);
      } catch (error) {
        console.error('Scan error:', error);
      }
    }
  };

  const processDocument = async (imageUri) => {
    try {
      // Use image crop picker for document edge detection
      const croppedImage = await ImagePicker.openCropper({
        path: imageUri,
        width: 800,
        height: 600,
        includeBase64: true,
      });

      setScannedDocument(croppedImage);
      Alert.alert('Success', 'Document scanned successfully!');
    } catch (error) {
      console.log('Processing error:', error);
    }
  };

  const selectFromGallery = () => {
    ImagePicker.openPicker({
      cropping: true,
      width: 800,
      height: 600,
      includeBase64: true,
    }).then(image => {
      setScannedDocument(image);
    }).catch(error => {
      console.log('Gallery selection error:', error);
    });
  };

  return (
    <View style={styles.container}>
      <RNCamera
        ref={cameraRef}
        style={styles.camera}
        type={RNCamera.Constants.Type.back}
        captureAudio={false}
      />

      <View style={styles.overlay}>
        <View style={styles.scanFrame} />
        <Text style={styles.instruction}>
          Position document within the frame
        </Text>
      </View>

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={selectFromGallery}>
          <Text style={styles.buttonText}>Gallery</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.scanButton} onPress={scanDocument}>
          <Text style={styles.scanButtonText}>Scan</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.button} onPress={() => setScannedDocument(null)}>
          <Text style={styles.buttonText}>Clear</Text>
        </TouchableOpacity>
      </View>

      {scannedDocument && (
        <View style={styles.result}>
          <Text style={styles.resultText}>Document scanned successfully!</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  camera: {
    flex: 1,
  },
  overlay: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: 'center',
    alignItems: 'center',
  },
  scanFrame: {
    width: 280,
    height: 200,
    borderWidth: 2,
    borderColor: '#007AFF',
    borderStyle: 'dashed',
    borderRadius: 10,
  },
  instruction: {
    marginTop: 20,
    color: 'white',
    fontSize: 16,
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 10,
    borderRadius: 5,
  },
  controls: {
    position: 'absolute',
    bottom: 30,
    left: 0,
    right: 0,
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
  },
  button: {
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
  },
  scanButton: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
  },
  scanButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  result: {
    position: 'absolute',
    top: 50,
    left: 20,
    right: 20,
    backgroundColor: 'rgba(0,255,0,0.8)',
    padding: 10,
    borderRadius: 5,
  },
  resultText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default DocumentScanner;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Camera Permissions**: Requesting and handling camera/storage permissions
- ✅ **Photo Capture**: Using image picker libraries for photo selection
- ✅ **Video Recording**: Recording and processing video content
- ✅ **Media Library**: Accessing device photo/video library
- ✅ **Real-time Camera**: Live camera preview with controls
- ✅ **Image Processing**: Resizing, compression, and manipulation
- ✅ **Advanced Features**: QR scanning, document scanning, custom controls

### **Best Practices**
1. **Always request permissions** before accessing camera/media
2. **Handle errors gracefully** with user-friendly messages
3. **Optimize image sizes** for better performance and storage
4. **Test on real devices** for accurate camera behavior
5. **Provide fallback options** when camera is unavailable
6. **Clean up resources** to prevent memory leaks

### **Next Steps**
- Experiment with different camera libraries
- Implement advanced image processing features
- Add AR capabilities using camera
- Create custom camera UI components

---

## 🎯 **Assignment**

### **Task 1: Photo Sharing App**
Create a photo sharing app with:
- Camera photo capture
- Gallery image selection
- Image filters and effects
- Photo upload functionality
- Social sharing features

### **Task 2: Video Recorder**
Build a video recording app that:
- Records high-quality video
- Provides recording controls (start/stop/pause)
- Shows recording duration
- Allows video playback
- Saves videos to device storage

### **Task 3: QR Code Scanner**
Implement a QR code scanner with:
- Real-time camera preview
- QR code detection and decoding
- History of scanned codes
- Actions based on QR content (URL, text, etc.)
- Error handling for invalid codes

---

## 📚 **Additional Resources**
- [React Native Camera Documentation](https://react-native-camera.github.io/react-native-camera/)
- [React Native Image Picker](https://github.com/react-native-image-picker/react-native-image-picker)
- [Expo Camera Documentation](https://docs.expo.dev/versions/latest/sdk/camera/)
- [React Native Image Crop Picker](https://github.com/ivpusic/react-native-image-crop-picker)
- [Camera Permission Guide](https://reactnative.dev/docs/permissions)

---

**Next Lesson**: [Lesson 16: Maps & Location Services](Lesson%2016_%20Maps%20&%20Location%20Services.md)