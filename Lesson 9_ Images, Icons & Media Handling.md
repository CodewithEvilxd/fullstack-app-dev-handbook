
# 🖼️ Lesson 9: Images, Icons & Media Handling

## 🎯 Learning Objectives
Is lesson ke baad aap:
- Images efficiently load aur display kar sakte hain
- Icons aur vector graphics use kar sakte hain
- Image picker aur camera integration kar sakte hain
- Media files handle kar sakte hain
- Image optimization techniques samjhenge

---

## 📖 Image Component Basics

React Native mein Image component media content display karne ke liye use hota hai.

### 1. Basic Image Usage

```javascript
import React from 'react';
import {
  View,
  Image,
  ScrollView,
  Text,
  StyleSheet,
  Dimensions
} from 'react-native';

const BasicImageExample = () => {
  const { width } = Dimensions.get('window');

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Image Examples</Text>
      
      {/* Local Image */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Local Images</Text>
        <Image
          source={require('../assets/logo.png')}
          style={styles.localImage}
          resizeMode="contain"
        />
        <Text style={styles.description}>
          Local images are bundled with your app and loaded from the assets folder.
        </Text>
      </View>

      {/* Network Image */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Network Images</Text>
        <Image
          source={{
            uri: 'https://picsum.photos/300/200?random=1',
          }}
          style={styles.networkImage}
          resizeMode="cover"
        />
        <Text style={styles.description}>
          Network images are loaded from URLs and cached automatically.
        </Text>
      </View>

      {/* Different Resize Modes */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Resize Modes</Text>
        
        {['cover', 'contain', 'stretch', 'repeat', 'center'].map(mode => (
          <View key={mode} style={styles.resizeModeContainer}>
            <Text style={styles.resizeModeTitle}>{mode}</Text>
            <Image
              source={{ uri: 'https://picsum.photos/400/300?random=2' }}
              style={styles.resizeModeImage}
              resizeMode={mode}
            />
          </View>
        ))}
      </View>

      {/* Responsive Images */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Responsive Images</Text>
        <Image
          source={{ uri: 'https://picsum.photos/800/600?random=3' }}
          style={[styles.responsiveImage, { width: width - 40 }]}
          resizeMode="cover"
        />
        <Text style={styles.description}>
          This image adapts to screen width while maintaining aspect ratio.
        </Text>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  section: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
  },
  description: {
    fontSize: 14,
    color: '#666',
    marginTop: 10,
    lineHeight: 20,
  },
  localImage: {
    width: 150,
    height: 150,
    alignSelf: 'center',
    borderRadius: 8,
  },
  networkImage: {
    width: '100%',
    height: 200,
    borderRadius: 8,
  },
  resizeModeContainer: {
    marginBottom: 20,
  },
  resizeModeTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF',
    marginBottom: 8,
  },
  resizeModeImage: {
    width: '100%',
    height: 120,
    backgroundColor: '#f0f0f0',
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  responsiveImage: {
    height: 200,
    borderRadius: 8,
  },
});

export default BasicImageExample;
```

### 2. Advanced Image Features

```javascript
import React, { useState } from 'react';
import {
  View,
  Image,
  Text,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
  Alert
} from 'react-native';

const AdvancedImageExample = () => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);
  const [imageUri, setImageUri] = useState('https://picsum.photos/300/300?random=4');

  const handleImageLoad = () => {
    setLoading(false);
    setError(false);
  };

  const handleImageError = () => {
    setLoading(false);
    setError(true);
    Alert.alert('Error', 'Failed to load image');
  };

  const changeImage = () => {
    setLoading(true);
    setError(false);
    const randomId = Math.floor(Math.random() * 1000);
    setImageUri(`https://picsum.photos/300/300?random=${randomId}`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Advanced Image Features</Text>
      
      <View style={styles.imageContainer}>
        {loading && (
          <View style={styles.loadingOverlay}>
            <ActivityIndicator size="large" color="#007AFF" />
            <Text style={styles.loadingText}>Loading image...</Text>
          </View>
        )}
        
        {error && (
          <View style={styles.errorOverlay}>
            <Text style={styles.errorText}>❌</Text>
            <Text style={styles.errorMessage}>Failed to load image</Text>
          </View>
        )}
        
        <Image
          source={{ uri: imageUri }}
          style={styles.image}
          onLoad={handleImageLoad}
          onError={handleImageError}
          resizeMode="cover"
        />
      </View>

      <TouchableOpacity style={styles.button} onPress={changeImage}>
        <Text style={styles.buttonText}>Load New Image</Text>
      </TouchableOpacity>

      {/* Image with Blur */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Blurred Image</Text>
        <Image
          source={{ uri: 'https://picsum.photos/300/200?random=5' }}
          style={styles.blurredImage}
          blurRadius={5}
          resizeMode="cover"
        />
      </View>

      {/* Image with Tint */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Tinted Image</Text>
        <Image
          source={{ uri: 'https://picsum.photos/300/200?random=6' }}
          style={styles.tintedImage}
          resizeMode="cover"
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  imageContainer: {
    position: 'relative',
    alignSelf: 'center',
    marginBottom: 20,
  },
  image: {
    width: 300,
    height: 300,
    borderRadius: 12,
  },
  loadingOverlay: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    backgroundColor: 'rgba(255,255,255,0.9)',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 12,
    zIndex: 1,
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorOverlay: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    backgroundColor: 'rgba(255,255,255,0.9)',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 12,
    zIndex: 1,
  },
  errorText: {
    fontSize: 48,
  },
  errorMessage: {
    marginTop: 10,
    fontSize: 16,
    color: '#FF3B30',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    paddingHorizontal: 30,
    borderRadius: 8,
    alignSelf: 'center',
    marginBottom: 30,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
  },
  blurredImage: {
    width: '100%',
    height: 150,
    borderRadius: 8,
  },
  tintedImage: {
    width: '100%',
    height: 150,
    borderRadius: 8,
    tintColor: '#007AFF',
  },
});
```

---

## 🎨 Icons and Vector Graphics

### 1. React Native Vector Icons

```bash
# Install vector icons
npm install react-native-vector-icons

# For iOS
cd ios && pod install

# For Android, add to android/app/build.gradle:
# apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
```

```javascript
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView
} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import FontAwesome from 'react-native-vector-icons/FontAwesome';
import Ionicons from 'react-native-vector-icons/Ionicons';
import AntDesign from 'react-native-vector-icons/AntDesign';

const IconsExample = () => {
  const iconSets = [
    {
      name: 'Material Icons',
      Component: Icon,
      icons: ['home', 'search', 'favorite', 'settings', 'person', 'notifications']
    },
    {
      name: 'Font Awesome',
      Component: FontAwesome,
      icons: ['heart', 'star', 'user', 'camera', 'music', 'shopping-cart']
    },
    {
      name: 'Ionicons',
      Component: Ionicons,
      icons: ['ios-home', 'ios-search', 'ios-heart', 'ios-settings', 'ios-person', 'ios-camera']
    },
    {
      name: 'Ant Design',
      Component: AntDesign,
      icons: ['home', 'search1', 'heart', 'setting', 'user', 'camera']
    }
  ];

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Vector Icons Examples</Text>
      
      {iconSets.map((iconSet, setIndex) => (
        <View key={setIndex} style={styles.section}>
          <Text style={styles.sectionTitle}>{iconSet.name}</Text>
          <View style={styles.iconGrid}>
            {iconSet.icons.map((iconName, iconIndex) => (
              <TouchableOpacity
                key={iconIndex}
                style={styles.iconItem}
                onPress={() => console.log(`Pressed ${iconName}`)}
              >
                <iconSet.Component
                  name={iconName}
                  size={30}
                  color="#007AFF"
                />
                <Text style={styles.iconName}>{iconName}</Text>
              </TouchableOpacity>
            ))}
          </View>
        </View>
      ))}

      {/* Icon Buttons */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Icon Buttons</Text>
        <View style={styles.buttonRow}>
          <TouchableOpacity style={[styles.iconButton, { backgroundColor: '#FF3B30' }]}>
            <FontAwesome name="heart" size={20} color="white" />
            <Text style={styles.iconButtonText}>Like</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={[styles.iconButton, { backgroundColor: '#007AFF' }]}>
            <Icon name="share" size={20} color="white" />
            <Text style={styles.iconButtonText}>Share</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={[styles.iconButton, { backgroundColor: '#28A745' }]}>
            <Ionicons name="ios-download" size={20} color="white" />
            <Text style={styles.iconButtonText}>Download</Text>
          </TouchableOpacity>
        </View>
      </View>

      {/* Different Icon Sizes */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Icon Sizes</Text>
        <View style={styles.sizeRow}>
          <Icon name="star" size={16} color="#FFD700" />
          <Icon name="star" size={24} color="#FFD700" />
          <Icon name="star" size={32} color="#FFD700" />
          <Icon name="star" size={48} color="#FFD700" />
          <Icon name="star" size={64} color="#FFD700" />
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  section: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
  },
  iconGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-around',
  },
  iconItem: {
    alignItems: 'center',
    padding: 15,
    margin: 5,
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
    minWidth: 80,
  },
  iconName: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
    textAlign: 'center',
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  iconButton: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 25,
  },
  iconButtonText: {
    color: 'white',
    marginLeft: 8,
    fontSize: 16,
    fontWeight: 'bold',
  },
  sizeRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
  },
});

export default IconsExample;
```

---

## 📷 Image Picker and Camera

### 1. Image Picker Setup

```bash
# Install image picker
npm install react-native-image-picker

# For iOS permissions (ios/YourApp/Info.plist)
<key>NSCameraUsageDescription</key>
<string>This app needs access to camera to take photos</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>This app needs access to photo library to select images</string>

# For Android permissions (android/app/src/main/AndroidManifest.xml)
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  Image,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ScrollView
} from 'react-native';
import { launchImageLibrary, launchCamera } from 'react-native-image-picker';

const ImagePickerExample = () => {
  const [selectedImages, setSelectedImages] = useState([]);

  const imagePickerOptions = {
    mediaType: 'photo',
    includeBase64: false,
    maxHeight: 2000,
    maxWidth: 2000,
    quality: 0.8,
  };

  const selectImageFromLibrary = () => {
    launchImageLibrary(imagePickerOptions, (response) => {
      if (response.didCancel) {
        console.log('User cancelled image picker');
      } else if (response.error) {
        console.log('ImagePicker Error: ', response.error);
        Alert.alert('Error', 'Failed to select image');
      } else if (response.assets && response.assets[0]) {
        const imageUri = response.assets[0].uri;
        setSelectedImages(prev => [...prev, {
          id: Date.now().toString(),
          uri: imageUri,
          name: response.assets[0].fileName || 'image.jpg',
          size: response.assets[0].fileSize,
          type: response.assets[0].type
        }]);
      }
    });
  };

  const takePhotoWithCamera = () => {
    launchCamera(imagePickerOptions, (response) => {
      if (response.didCancel) {
        console.log('User cancelled camera');
      } else if (response.error) {
        console.log('Camera Error: ', response.error);
        Alert.alert('Error', 'Failed to take photo');
      } else if (response.assets && response.assets[0]) {
        const imageUri = response.assets[0].uri;
        setSelectedImages(prev => [...prev, {
          id: Date.now().toString(),
          uri: imageUri,
          name: response.assets[0].fileName || 'photo.jpg',
          size: response.assets[0].fileSize,
          type: response.assets[0].type
        }]);
      }
    });
  };

  const removeImage = (imageId) => {
    setSelectedImages(prev => prev.filter(img => img.id !== imageId));
  };

  const clearAllImages = () => {
    Alert.alert(
      'Clear All Images',
      'Are you sure you want to remove all images?',
      [
        { text: 'Cancel', style: 'cancel' },
        { text: 'Clear', style: 'destructive', onPress: () => setSelectedImages([]) }
      ]
    );
  };

  const formatFileSize = (bytes) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Image Picker Example</Text>
      
      {/* Action Buttons */}
      <View style={styles.buttonContainer}>
        <TouchableOpacity style={styles.button} onPress={selectImageFromLibrary}>
          <Text style={styles.buttonText}>📱 Select from Library</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={[styles.button, styles.cameraButton]} onPress={takePhotoWithCamera}>
          <Text style={styles.buttonText}>📷 Take Photo</Text>
        </TouchableOpacity>
        
        {selectedImages.length > 0 && (
          <TouchableOpacity style={[styles.button, styles.clearButton]} onPress={clearAllImages}>
            <Text style={styles.buttonText}>🗑️ Clear All</Text>
          </TouchableOpacity>
        )}
      </View>

      {/* Selected Images */}
      {selectedImages.length > 0 && (
        <View style={styles.imagesSection}>
          <Text style={styles.sectionTitle}>Selected Images ({selectedImages.length})</Text>
          
          {selectedImages.map((image, index) => (
            <View key={image.id} style={styles.imageItem}>
              <Image source={{ uri: image.uri }} style={styles.selectedImage} />
              
              <View style={styles.imageInfo}>
                <Text style={styles.imageName}>{image.name}</Text>
                <Text style={styles.imageSize}>{formatFileSize(image.size)}</Text>
                <Text style={styles.imageType}>{image.type}</Text>
              </View>
              
              <TouchableOpacity
                style={styles.removeButton}
                onPress={() => removeImage(image.id)}
              >
                <Text style={styles.removeButtonText}>×</Text>
              </TouchableOpacity>
            </View>
          ))}
        </View>
      )}

      {/* Empty State */}
      {selectedImages.length === 0 && (
        <View style={styles.emptyState}>
          <Text style={styles.emptyStateText}>📸</Text>
          <Text style={styles.emptyStateTitle}>No Images Selected</Text>
          <Text style={styles.emptyStateDescription}>
            Use the buttons above to select images from your library or take a new photo
          </Text>
        </View>
      )}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  buttonContainer: {
    padding: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 15,
  },
  cameraButton: {
    backgroundColor: '#28A745',
  },
  clearButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  imagesSection: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
  },
  imageItem: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  selectedImage: {
    width: 60,
    height: 60,
    borderRadius: 8,
    marginRight: 15,
  },
  imageInfo: {
    flex: 1,
  },
  imageName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 2,
  },
  imageSize: {
    fontSize: 14,
    color: '#666',
    marginBottom: 2,
  },
  imageType: {
    fontSize: 12,
    color: '#999',
  },
  removeButton: {
    backgroundColor: '#FF3B30',
    width: 30,
    height: 30,
    borderRadius: 15,
    justifyContent: 'center',
    alignItems: 'center',
  },
  removeButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  emptyState: {
    alignItems: 'center',
    padding: 50,
    backgroundColor: 'white',
    margin: 15,
    borderRadius: 12,
  },
  emptyStateText: {
    fontSize: 64,
    marginBottom: 20,
  },
  emptyStateTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  emptyStateDescription: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    lineHeight: 24,
  },
});

export default ImagePickerExample;
```

---

## 🎬 Video and Media Handling

### 1. Video Player

```bash
# Install video player
npm install react-native-video
```

```javascript
import React, { useState, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Dimensions,
  Alert
} from 'react-native';
import Video from 'react-native-video';

const VideoPlayerExample = () => {
  const videoRef = useRef(null);
  const [paused, setPaused] = useState(true);
  const [currentTime, setCurrentTime] = useState(0);
  const [duration, setDuration] = useState(0);
  const [loading, setLoading] = useState(true);

  const { width } = Dimensions.get('window');
  const videoHeight = (width * 9) / 16; // 16:9 aspect ratio

  const videoSources = [
    {
      id: 1,
      title: 'Sample Video 1',
      uri: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4'
    },
    {
      id: 2,
      title: 'Sample Video 2',
      uri: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4'
    }
  ];

  const [currentVideo, setCurrentVideo] = useState(videoSources[0]);

  const onLoad = (data) => {
    setDuration(data.duration);
    setLoading(false);
  };

  const onProgress = (data) => {
    setCurrentTime(data.currentTime);
  };

  const onError = (error) => {
    console.log('Video Error:', error);
    Alert.alert('Error', 'Failed to load video');
    setLoading(false);
  };

  const togglePlayPause = () => {
    setPaused(!paused);
  };

  const seekTo = (time) => {
    videoRef.current?.seek(time);
  };

  const formatTime = (timeInSeconds) => {
    const minutes = Math.floor(timeInSeconds / 60);
    const seconds = Math.floor(timeInSeconds % 60);
    return `${minutes}:${seconds.toString().padStart(2, '0')}`;
  };

  const changeVideo = (video) => {
    setCurrentVideo(video);
    setPaused(true);
    setCurrentTime(0);
    setLoading(true);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Video Player Example</Text>
      
      {/* Video Player */}
      <View style={styles.videoContainer}>
        <Video
          ref={videoRef}
          source={{ uri: currentVideo.uri }}
          style={[styles.video, { width, height: videoHeight }]}
          paused={paused}
          onLoad={onLoad}
          onProgress={onProgress}
          onError={onError}
          resizeMode="contain"
          controls={false}
        />
        
        {loading && (
          <View style={styles.loadingOverlay}>
            <Text style={styles.loadingText}>Loading video...</Text>
          </View>
        )}
      </View>

      {/* Video Info */}
      <View style={styles.videoInfo}>
        <Text style={styles.videoTitle}>{currentVideo.title}</Text>
        <Text style={styles.videoTime}>
          {formatTime(currentTime)} / {formatTime(duration)}
        </Text>
      </View>

      {/* Controls */}
      <View style={styles.controls}>
        <TouchableOpacity
          style={styles.controlButton}
          onPress={() => seekTo(Math.max(0, currentTime - 10))}
        >
          <Text style={styles.controlButtonText}>⏪ -10s</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.controlButton, styles.playButton]}
          onPress={togglePlayPause}
        >
          <Text style={styles.controlButtonText}>
            {paused ? '▶️ Play' : '⏸️ Pause'}
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.controlButton}
          onPress={() => seekTo(Math.min(duration, currentTime + 10))}
        >
          <Text style={styles.controlButtonText}>⏩ +10s</Text>
        </TouchableOpacity>
      </View>

      {/* Progress Bar */}
      <View style={styles.progressContainer}>
        <View style={styles.progressBar}>
          <View
            style={[
              styles.progressFill,
              { width: `${(currentTime / duration) * 100}%` }
            ]}
          />
        </View>
      </View>

      {/* Video Selection */}
      <View style={styles.videoSelection}>
        <Text style={styles.selectionTitle}>Select Video:</Text>
        {videoSources.map(video => (
          <TouchableOpacity
            key={video.id}
            style={[
              styles.videoOption,
              currentVideo.id === video.id && styles.selectedVideo
            ]}
            onPress={() => changeVideo(video)}
          >
            <Text style={[
              styles.videoOptionText,
              currentVideo.id === video.id && styles.selectedVideoText
            ]}>
              {video.title}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 15,
    backgroundColor: '#333',
    color: 'white',
  },
  videoContainer: {
    position: 'relative',
  },
  video: {
    backgroundColor: '#000',
  },
  loadingOverlay: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    backgroundColor: 'rgba(0,0,0,0.7)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    color: 'white',
    fontSize: 16,
  },
  videoInfo: {
    backgroundColor: '#333',
    padding: 15,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  videoTitle: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
    flex: 1,
  },
  videoTime: {
    color: '#ccc',
    fontSize: 14,
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    backgroundColor: '#333',
    paddingVertical: 15,
  },
  controlButton: {
    backgroundColor: '#555',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderRadius: 5,
  },
  playButton: {
    backgroundColor: '#007AFF',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  progressContainer: {
    backgroundColor: '#333',
    paddingHorizontal: 15,
    paddingVertical: 10,
  },
  progressBar: {
    height: 4,
    backgroundColor: '#555',
    borderRadius: 2,
    overflow: 'hidden',
  },
  progressFill: {
    height: '100%',
    backgroundColor: '#007AFF',
  },
  videoSelection: {
    backgroundColor: '#222',
    padding: 15,
  },
  selectionTitle: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  videoOption: {
    backgroundColor: '#444',
    padding: 12,
    borderRadius: 5,
    marginBottom: 8,
  },
  selectedVideo: {
    backgroundColor: '#007AFF',
  },
  videoOptionText: {
    color: 'white',
    fontSize: 14,
  },
  selectedVideoText: {
    fontWeight: 'bold',
  },
});

export default VideoPlayerExample;
```

---

## 🎵 Audio Handling

### 1. Audio Player

```bash
# Install audio player
npm install react-native-sound
npm install react-native-track-player
```

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ScrollView
} from 'react-native';
import Sound from 'react-native-sound';

const AudioPlayerExample = () => {
  const [sound, setSound] = useState(null);
  const [isPlaying, setIsPlaying] = useState(false);
  const [currentTime, setCurrentTime] = useState(0);
  const [duration, setDuration] = useState(0);

  const audioFiles = [
    {
      id: 1,
      title: 'Sample Audio 1',
      url: 'https://www.soundjay.com/misc/sounds/bell-ringing-05.wav',
      artist: 'Sample Artist'
    },
    {
      id: 2,
      title: 'Sample Audio 2',
      url: 'https://www.soundjay.com/misc/sounds/fail-buzzer-02.wav',
      artist: 'Another Artist'
    }
  ];

  const [currentTrack, setCurrentTrack] = useState(audioFiles[0]);

  useEffect(() => {
    // Enable playback in silence mode
    Sound.setCategory('Playback');
    
    return () => {
      if (sound) {
        sound.release();
      }
    };
  }, [sound]);

  const loadAudio = (audioUrl) => {
    if (sound) {
      sound.stop();
      sound.release();
    }

    const newSound = new Sound(audioUrl, null, (error) => {
      if (error) {
        console.log('Failed to load sound', error);
        Alert.alert('Error', 'Failed to load audio');
        return;
      }
      
      setDuration(newSound.getDuration());
      setSound(newSound);
    });
  };

  const playPause = () => {
    if (!sound) {
      loadAudio(currentTrack.url);
      return;
    }

    if (isPlaying) {
      sound.pause();
      setIsPlaying(false);
    } else {
      sound.play((success) => {
        if (success) {
          console.log('Successfully finished playing');
          setIsPlaying(false);
          setCurrentTime(0);
        } else {
          console.log('Playback failed');
          Alert.alert('Error', 'Playback failed');
        }
      });
      setIsPlaying(true);
    }
  };

  const stop = () => {
    if (sound) {
      sound.stop();
      setIsPlaying(false);
      setCurrentTime(0);
    }
  };

  const changeTrack = (track) => {
    setCurrentTrack(track);
    setIsPlaying(false);
    setCurrentTime(0);
    loadAudio(track.url);
  };

  const formatTime = (timeInSeconds) => {
    const minutes = Math.floor(timeInSeconds / 60);
    const seconds = Math.floor(timeInSeconds % 60);
    return `${minutes}:${seconds.toString().padStart(2, '0')}`;
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Audio Player Example</Text>
      
      {/* Current Track Info */}
      <View style={styles.trackInfo}>
        <Text style={styles.trackTitle}>{currentTrack.title}</Text>
        <Text style={styles.trackArtist}>{currentTrack.artist}</Text>
        <Text style={styles.trackTime}>
          {formatTime(currentTime)} / {formatTime(duration)}
        </Text>
      </View>

      {/* Audio Controls */}
      <View style={styles.controls}>
        <TouchableOpacity style={styles.controlButton} onPress={stop}>
          <Text style={styles.controlButtonText}>⏹️ Stop</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.controlButton, styles.playButton]}
          onPress={playPause}
        >
          <Text style={styles.controlButtonText}>
            {isPlaying ? '⏸️ Pause' : '▶️ Play'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Track Selection */}
      <View style={styles.trackList}>
        <Text style={styles.sectionTitle}>Select Track:</Text>
        {audioFiles.map(track => (
          <TouchableOpacity
            key={track.id}
            style={[
              styles.trackItem,
              currentTrack.id === track.id && styles.selectedTrack
            ]}
            onPress={() => changeTrack(track)}
          >
            <Text style={[
              styles.trackItemTitle,
              currentTrack.id === track.id && styles.selectedTrackText
            ]}>
              {track.title}
            </Text>
            <Text style={[
              styles.trackItemArtist,
              currentTrack.id === track.id && styles.selectedTrackText
            ]}>
              {track.artist}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  trackInfo: {
    backgroundColor: 'white',
    padding: 20,
    margin: 15,
    borderRadius: 12,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  trackTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  trackArtist: {
    fontSize: 16,
    color: '#666',
    marginBottom: 10,
  },
  trackTime: {
    fontSize: 14,
    color: '#999',
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'center',
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  controlButton: {
    backgroundColor: '#6C757D',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 25,
    marginHorizontal: 10,
  },
  playButton: {
    backgroundColor: '#007AFF',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  trackList: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
  },
  trackItem: {
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  selectedTrack: {
    backgroundColor: '#007AFF',
  },
  trackItemTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 3,
  },
  trackItemArtist: {
    fontSize: 14,
    color: '#666',
  },
  selectedTrackText: {
    color: 'white',
  },
});

export default AudioPlayerExample;
```

---

## 📝 Assignment 9: Media Handling

### Task 1: Image Gallery App (120 minutes)
**Objective:** Create a complete image gallery with picker integration

**Requirements:**
1. **Image Selection:**
   - Camera capture
   - Gallery selection
   - Multiple image selection
   - Image preview

2. **Gallery Features:**
   - Grid layout display
   - Full-screen image view
   - Image deletion
   - Image sharing

3. **Image Management:**
   - Local storage
   - Image compression
   - File size display
   - Image metadata

**Expected Features:**
- Responsive grid layout
- Smooth transitions
- Loading states
- Error handling
- Empty state design

### Task 2: Media Player App (150 minutes)
**Objective:** Build a complete media player

**Requirements:**
1. **Video Player:**
   - Play/pause controls
   - Seek functionality
   - Progress bar
   - Full-screen mode

2. **Audio Player:**
   - Background playback
   - Playlist management
   - Track information display
   - Control notifications

3. **Media Library:**
   - Local media scanning
   - Playlist creation
   - Search functionality
   - Favorites system

### Task 3: Icon Library Component (90 minutes)
**Objective:** Create reusable icon components

**Requirements:**
1. **Icon Component:**
   - Multiple icon sets support
   - Size variations
   - Color customization
   - Touch feedback

2. **Icon Browser:**
   - Searchable icon list
   - Category filtering
   - Copy icon name
   - Usage examples

---

## 🧠 Quiz 9: Images & Media (20 Questions)

### Multiple Choice Questions (1-12)

**Q1. Image component mein resizeMode ki default value kya hai?**
a) cover
b) contain
c) stretch
d) center

**Q2. Network images ke liye source prop mein kya provide karna hota hai?**
a) require() statement
b) Object with uri property
c) String path
d) Base64 string

**Q3. Image loading state handle karne ke liye kaunsa event use karte hain?**
a) onLoad
b) onLoadStart
c) onLoadEnd
d) All of the above

**Q4. Vector icons install karne ke baad Android mein kya karna padta hai?**
a) Nothing
b) Add fonts.gradle
c) Rebuild app
d) Both b and c

**Q5. Image picker permissions iOS mein kahan set karte hain?**
a) AndroidManifest.xml
b) Info.plist
c) package.json
d) metro.config.js

**Q6. Video component mein controls={false} ka matlab kya hai?**
a) Video play nahi hoga
b) Custom controls banane honge
c) Audio mute hoga
d) Video loop nahi hoga

**Q7. Audio playback ke liye kaunsa category set karna chahiye?**
a) 'Record'
b) 'Playback'
c) 'Ambient'
d) 'SoloAmbient'

**Q8. Image blur effect ke liye kaunsi property use karte hain?**
a) blur
b) blurRadius
c) blurEffect
d) imageBlur

**Q9. Multiple images select karne ke liye image picker mein kya set karna hota hai?**
a) multiple: true
b) selectionLimit: 0
c) maxFiles: 10
d) allowsMultipleSelection: true

**Q10. Video progress track karne ke liye kaunsa event use karte hain?**
a) onProgress
b) onTimeUpdate
c) onSeek
d) onCurrentTime

**Q11. Image tint color apply karne ke liye kaunsi property use karte hain?**
a) tint
b) tintColor
c) imageColor
d) colorFilter

**Q12. Audio file release karna kyu zaroori hai?**
a) Memory leak prevent karne ke liye
b) Performance improve karne ke liye
c) Battery save karne ke liye
d) All of the above

### True/False Questions (13-16)

**Q13. Local images automatically cached hote hain.** (True/False)

**Q14. Vector icons scalable hote hain without quality loss.** (True/False)

**Q15. Video component sirf network URLs support karta hai.** (True/False)

**Q16. Image picker camera aur gallery dono access kar sakta hai.** (True/False)

### Short Answer Questions (17-20)

**Q17. Image loading performance optimize karne ke 3 tarike batao.**

**Q18. Video player mein custom controls banane ke fayde kya hain?**

**Q19. Audio background playback implement karne ke steps batao.**

**Q20. Image picker permissions handle karne ka proper tarika kya hai?**

---

## 🎯 Quiz Answers

**MCQ Answers:**
1. b) contain
2. b) Object with uri property
3. d) All of the above
4. d) Both b and c
5. b) Info.plist
6. b) Custom controls banane honge
7. b) 'Playback'
8. b) blurRadius
9. b) selectionLimit: 0
10. a) onProgress
11. b) tintColor
12. d) All of the above

**True/False Answers:**
13. True
14. True
15. False (local files bhi support karta hai)
16. True

**Short Answers:**
17. Image caching, proper resizeMode, image compression
18. Better UX control, custom styling, specific functionality
19. Background mode setup, audio session configuration, notification controls
20. Permission request, error handling, fallback options

---

## 🔗 Additional Resources

### Libraries
- [React Native Image Picker](https://github.com/react-native-image-picker/react-native-image-picker)
- [React Native Video](https://github.com/react-native-video/react-native-video)
- [React Native Vector Icons](https://github.com/oblador/react-native-vector-icons)
- [React Native Sound](https://github.com/zmxv/react-native-sound)

### Documentation
- [Image Component Docs](https://reactnative.dev/docs/image)
- [Media Handling Guide](https://reactnative.dev/docs/handling-touches)
- [Platform Specific Code](https://reactnative.dev/docs/platform-specific-code)

---

## 🎉 Lesson Complete!

Congratulations! Aapne successfully:
- ✅ Image component aur advanced features sikhe
- ✅ Vector icons integration kiya
- ✅ Image picker aur camera use kiya
- ✅ Video aur audio playback implement kiya
- ✅ Media handling best practices samjhe

**Next Lesson Preview:** Debugging & Development Tools - React Native apps ko debug karne aur development workflow optimize karne ke techniques.

---

*Happy Coding! 🚀*