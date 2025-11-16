# Expo Mobile Development Guide

## Overview

Expo is a comprehensive platform built on top of React Native that provides a streamlined development experience with managed workflows, built-in libraries, and easy deployment. It eliminates much of the native configuration complexity while still allowing access to native features.

**Key Features:**
- **Managed Workflow**: No need to configure Xcode or Android Studio
- **Expo Go App**: Test on real devices instantly
- **Built-in Libraries**: Camera, location, notifications, and 50+ more
- **OTA Updates**: Push updates without app store approval
- **EAS (Expo Application Services)**: Cloud build and submit
- **File-Based Routing**: Expo Router for navigation
- **TypeScript Support**: Full type safety

**When to Use:**
- Rapid mobile app development
- Don't want to deal with native configuration
- Need common features (camera, maps, notifications)
- Want over-the-air updates
- Building MVP or prototype quickly
- Team without native development experience

**When Not to Use:**
- Need very specific native modules not in Expo
- Require deep native customization
- Building complex Bluetooth or background task apps
- Need absolute smallest app bundle size

## Installation

**📚 Official Documentation**: [Expo - Create a Project](https://expo.dev/get-started/create-a-project)

Follow the official setup guide for the latest installation instructions and template options.

### Quick Reference

**Create New Project:**
Expo provides templates including blank, TypeScript, tabs, and more.

**Run on Device:**
- **Expo Go**: Instant testing on real devices (scan QR code)
- **Development Build**: Custom native code support

**Setup Process:**
1. Create project with Expo CLI
2. Choose template (blank, tabs, etc.)
3. Start development server
4. Test on Expo Go or emulator

**Note**: CLI commands and templates may change. Always refer to the official docs above for the current setup method.

**Project Structure:**
```
my-app/
├── app/                    # Expo Router screens (file-based routing)
│   ├── (tabs)/            # Tab navigator group
│   │   ├── index.tsx      # Home tab
│   │   ├── explore.tsx    # Explore tab
│   │   └── _layout.tsx    # Tab layout
│   ├── [id].tsx           # Dynamic route
│   ├── _layout.tsx        # Root layout
│   └── +not-found.tsx     # 404 page
├── components/            # React components
├── constants/            # Constants and config
├── hooks/                # Custom hooks
├── assets/               # Images, fonts, etc.
├── app.json              # Expo configuration
├── package.json
└── tsconfig.json
```

### Run on Device

**Option 1: Expo Go (Easiest)**
```bash
npx expo start

# Scan QR code with:
# - iOS: Camera app
# - Android: Expo Go app
```

**Option 2: Development Build**
```bash
# Build development version with custom native code
npx expo run:ios
npx expo run:android
```

## Expo Router (File-Based Routing)

### Basic Routes

**app/_layout.tsx (Root Layout)**
```typescript
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack
      screenOptions={{
        headerStyle: {
          backgroundColor: '#007AFF',
        },
        headerTintColor: '#fff',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="about" options={{ title: 'About' }} />
      <Stack.Screen name="[id]" options={{ title: 'Details' }} />
    </Stack>
  );
}
```

**app/index.tsx (Home Screen)**
```typescript
import { View, Text, StyleSheet } from 'react-native';
import { Link } from 'expo-router';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Expo!</Text>

      <Link href="/about" style={styles.link}>
        Go to About
      </Link>

      <Link href="/user/123" style={styles.link}>
        View User 123
      </Link>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  link: {
    fontSize: 16,
    color: '#007AFF',
    marginTop: 12,
  },
});
```

**app/about.tsx**
```typescript
import { View, Text, StyleSheet } from 'react-native';

export default function AboutScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>About Screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
});
```

**app/[id].tsx (Dynamic Route)**
```typescript
import { View, Text, StyleSheet } from 'react-native';
import { useLocalSearchParams } from 'expo-router';

export default function DetailScreen() {
  const { id } = useLocalSearchParams();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Details for ID: {id}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 20,
  },
});
```

### Tab Navigator

**app/(tabs)/_layout.tsx**
```typescript
import { Tabs } from 'expo-router';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        headerShown: false,
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>🏠</Text>
          ),
        }}
      />
      <Tabs.Screen
        name="explore"
        options={{
          title: 'Explore',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>🔍</Text>
          ),
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>👤</Text>
          ),
        }}
      />
    </Tabs>
  );
}
```

### Programmatic Navigation

```typescript
import { router } from 'expo-router';
import { Pressable, Text } from 'react-native';

export function NavigationExample() {
  return (
    <Pressable onPress={() => router.push('/about')}>
      <Text>Go to About</Text>
    </Pressable>
  );
}

// With params
router.push({
  pathname: '/user/[id]',
  params: { id: '123' },
});

// Navigate back
router.back();

// Replace (don't add to history)
router.replace('/home');
```

## Built-in Features

### Camera

```bash
npx expo install expo-camera
```

```typescript
import { useState } from 'react';
import { View, Text, StyleSheet, Pressable } from 'react-native';
import { CameraView, CameraType, useCameraPermissions } from 'expo-camera';

export function CameraScreen() {
  const [facing, setFacing] = useState<CameraType>('back');
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission) {
    return <View />;
  }

  if (!permission.granted) {
    return (
      <View style={styles.container}>
        <Text style={styles.message}>
          We need your permission to show the camera
        </Text>
        <Pressable onPress={requestPermission} style={styles.button}>
          <Text style={styles.buttonText}>Grant Permission</Text>
        </Pressable>
      </View>
    );
  }

  function toggleCameraFacing() {
    setFacing((current) => (current === 'back' ? 'front' : 'back'));
  }

  return (
    <View style={styles.container}>
      <CameraView style={styles.camera} facing={facing}>
        <View style={styles.buttonContainer}>
          <Pressable style={styles.button} onPress={toggleCameraFacing}>
            <Text style={styles.buttonText}>Flip Camera</Text>
          </Pressable>
        </View>
      </CameraView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  message: {
    textAlign: 'center',
    paddingBottom: 10,
  },
  camera: {
    flex: 1,
  },
  buttonContainer: {
    flex: 1,
    flexDirection: 'row',
    backgroundColor: 'transparent',
    margin: 64,
  },
  button: {
    flex: 1,
    alignSelf: 'flex-end',
    alignItems: 'center',
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
  },
  buttonText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
  },
});
```

### Location

```bash
npx expo install expo-location
```

```typescript
import { useState, useEffect } from 'react';
import { View, Text, StyleSheet, Pressable } from 'react-native';
import * as Location from 'expo-location';

export function LocationScreen() {
  const [location, setLocation] = useState<Location.LocationObject | null>(null);
  const [errorMsg, setErrorMsg] = useState<string | null>(null);

  const getLocation = async () => {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();

      if (status !== 'granted') {
        setErrorMsg('Permission to access location was denied');
        return;
      }

      const location = await Location.getCurrentPositionAsync({});
      setLocation(location);
    } catch (error) {
      setErrorMsg('Error getting location');
    }
  };

  return (
    <View style={styles.container}>
      <Pressable style={styles.button} onPress={getLocation}>
        <Text style={styles.buttonText}>Get Location</Text>
      </Pressable>

      {errorMsg && <Text style={styles.error}>{errorMsg}</Text>}

      {location && (
        <View style={styles.locationInfo}>
          <Text>Latitude: {location.coords.latitude}</Text>
          <Text>Longitude: {location.coords.longitude}</Text>
          <Text>Altitude: {location.coords.altitude}</Text>
          <Text>Accuracy: {location.coords.accuracy}</Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  error: {
    color: 'red',
    marginTop: 20,
  },
  locationInfo: {
    marginTop: 20,
  },
});
```

### Notifications

```bash
npx expo install expo-notifications
```

**Setup:**

**app.json**
```json
{
  "expo": {
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ]
  }
}
```

**Usage:**
```typescript
import { useState, useEffect, useRef } from 'react';
import { View, Text, Pressable, StyleSheet } from 'react-native';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export function NotificationsScreen() {
  const [expoPushToken, setExpoPushToken] = useState<string>('');
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();

  useEffect(() => {
    registerForPushNotificationsAsync().then((token) => {
      if (token) setExpoPushToken(token);
    });

    notificationListener.current = Notifications.addNotificationReceivedListener(
      (notification) => {
        console.log('Notification received:', notification);
      }
    );

    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        console.log('Notification response:', response);
      }
    );

    return () => {
      if (notificationListener.current) {
        Notifications.removeNotificationSubscription(notificationListener.current);
      }
      if (responseListener.current) {
        Notifications.removeNotificationSubscription(responseListener.current);
      }
    };
  }, []);

  const schedulePushNotification = async () => {
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "You've got mail! 📬",
        body: 'Here is the notification body',
        data: { data: 'goes here' },
      },
      trigger: { seconds: 2 },
    });
  };

  return (
    <View style={styles.container}>
      <Text>Your push token: {expoPushToken}</Text>

      <Pressable style={styles.button} onPress={schedulePushNotification}>
        <Text style={styles.buttonText}>Schedule Notification</Text>
      </Pressable>
    </View>
  );
}

async function registerForPushNotificationsAsync() {
  let token;

  if (Device.isDevice) {
    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;

    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }

    if (finalStatus !== 'granted') {
      alert('Failed to get push token for push notification!');
      return;
    }

    token = (await Notifications.getExpoPushTokenAsync()).data;
  }

  return token;
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    marginTop: 20,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

### Image Picker

```bash
npx expo install expo-image-picker
```

```typescript
import { useState } from 'react';
import { View, Image, Pressable, Text, StyleSheet } from 'react-native';
import * as ImagePicker from 'expo-image-picker';

export function ImagePickerScreen() {
  const [image, setImage] = useState<string | null>(null);

  const pickImage = async () => {
    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsEditing: true,
      aspect: [4, 3],
      quality: 1,
    });

    if (!result.canceled) {
      setImage(result.assets[0].uri);
    }
  };

  const takePhoto = async () => {
    const { status } = await ImagePicker.requestCameraPermissionsAsync();

    if (status !== 'granted') {
      alert('Sorry, we need camera permissions!');
      return;
    }

    const result = await ImagePicker.launchCameraAsync({
      allowsEditing: true,
      aspect: [4, 3],
      quality: 1,
    });

    if (!result.canceled) {
      setImage(result.assets[0].uri);
    }
  };

  return (
    <View style={styles.container}>
      {image && <Image source={{ uri: image }} style={styles.image} />}

      <Pressable style={styles.button} onPress={pickImage}>
        <Text style={styles.buttonText}>Pick from Gallery</Text>
      </Pressable>

      <Pressable style={styles.button} onPress={takePhoto}>
        <Text style={styles.buttonText}>Take Photo</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
  },
  image: {
    width: '100%',
    height: 300,
    borderRadius: 8,
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 12,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

### Secure Storage

```bash
npx expo install expo-secure-store
```

```typescript
import * as SecureStore from 'expo-secure-store';

// Save
async function save(key: string, value: string) {
  await SecureStore.setItemAsync(key, value);
}

// Get
async function getValueFor(key: string) {
  const result = await SecureStore.getItemAsync(key);
  return result;
}

// Delete
async function deleteValue(key: string) {
  await SecureStore.deleteItemAsync(key);
}

// Example: Save auth token
await save('authToken', 'user_token_here');

// Retrieve auth token
const token = await getValueFor('authToken');
```

## API Integration

### Fetch with Error Handling

**services/api.ts**
```typescript
const API_BASE_URL = 'https://api.example.com';

export class APIError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public response?: any
  ) {
    super(message);
    this.name = 'APIError';
  }
}

async function request<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  try {
    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    const data = await response.json();

    if (!response.ok) {
      throw new APIError(
        data.message || 'Request failed',
        response.status,
        data
      );
    }

    return data;
  } catch (error) {
    if (error instanceof APIError) {
      throw error;
    }
    throw new Error('Network request failed');
  }
}

export const api = {
  get: <T>(endpoint: string) => request<T>(endpoint),

  post: <T>(endpoint: string, body: any) =>
    request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(body),
    }),

  put: <T>(endpoint: string, body: any) =>
    request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(body),
    }),

  delete: <T>(endpoint: string) =>
    request<T>(endpoint, {
      method: 'DELETE',
    }),
};

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

export const fetchUsers = () => api.get<User[]>('/users');
export const createUser = (data: Omit<User, 'id'>) => api.post<User>('/users', data);
```

## Configuration

### app.json

```json
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.yourcompany.myapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-router",
      "expo-font"
    ],
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

### Environment Variables

```bash
npx expo install expo-constants
```

**app.config.ts**
```typescript
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: 'MyApp',
  slug: 'my-app',
  extra: {
    apiUrl: process.env.API_URL || 'https://api.example.com',
    apiKey: process.env.API_KEY,
  },
});
```

**Usage:**
```typescript
import Constants from 'expo-constants';

const API_URL = Constants.expoConfig?.extra?.apiUrl;
const API_KEY = Constants.expoConfig?.extra?.apiKey;
```

## Building & Deployment

### EAS Build (Cloud Build Service)

**Setup:**
```bash
npm install -g eas-cli
eas login
eas build:configure
```

**eas.json**
```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "production": {}
  }
}
```

**Build Commands:**
```bash
# Build for iOS
eas build --platform ios

# Build for Android
eas build --platform android

# Build for both
eas build --platform all

# Development build
eas build --profile development --platform ios
```

### EAS Submit

```bash
# Submit to App Store
eas submit --platform ios

# Submit to Google Play
eas submit --platform android
```

### Over-The-Air (OTA) Updates

```bash
# Install
npx expo install expo-updates

# Publish update
eas update --branch production --message "Bug fixes"

# Create update for specific channel
eas update --branch production --platform ios
```

## Testing

### Jest Setup

```bash
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native
```

**jest.config.js**
```javascript
module.exports = {
  preset: 'jest-expo',
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg)',
  ],
};
```

**Example Test:**
```typescript
import { render, fireEvent } from '@testing-library/react-native';
import { HomeScreen } from '../app/index';

describe('HomeScreen', () => {
  it('renders correctly', () => {
    const { getByText } = render(<HomeScreen />);
    expect(getByText('Welcome to Expo!')).toBeTruthy();
  });

  it('navigates on button press', () => {
    const { getByText } = render(<HomeScreen />);
    const button = getByText('Go to About');

    fireEvent.press(button);
    // Assert navigation happened
  });
});
```

## Best Practices

### 1. Use Expo Router for Navigation

```typescript
// Good: File-based routing
import { Link } from 'expo-router';
<Link href="/profile">Profile</Link>

// Also good: Programmatic navigation
import { router } from 'expo-router';
router.push('/profile');
```

### 2. Lazy Load Screens

```typescript
import { lazy, Suspense } from 'react';
import { ActivityIndicator } from 'react-native';

const ProfileScreen = lazy(() => import('./screens/ProfileScreen'));

export default function App() {
  return (
    <Suspense fallback={<ActivityIndicator />}>
      <ProfileScreen />
    </Suspense>
  );
}
```

### 3. Use Expo's Asset Loading

```typescript
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';
import { useEffect } from 'react';

SplashScreen.preventAutoHideAsync();

export default function App() {
  const [loaded, error] = useFonts({
    'SpaceMono': require('./assets/fonts/SpaceMono-Regular.ttf'),
  });

  useEffect(() => {
    if (loaded || error) {
      SplashScreen.hideAsync();
    }
  }, [loaded, error]);

  if (!loaded && !error) {
    return null;
  }

  return <YourApp />;
}
```

### 4. Handle Permissions Gracefully

```typescript
const { status } = await Location.requestForegroundPermissionsAsync();

if (status !== 'granted') {
  Alert.alert(
    'Permission Required',
    'This app needs location access to function properly.',
    [
      { text: 'Cancel', style: 'cancel' },
      { text: 'Open Settings', onPress: () => Linking.openSettings() },
    ]
  );
  return;
}
```

## Debugging

### Expo Dev Tools

```bash
npx expo start

# Press in terminal:
# - i: Open iOS simulator
# - a: Open Android emulator
# - w: Open web browser
# - r: Reload app
# - m: Toggle menu
```

### React DevTools

```bash
npm install -g react-devtools
react-devtools
```

### Flipper (for Development Builds)

Built-in when using development builds:
- Network inspector
- Layout inspector
- Crash reporter
- Logs viewer

## Resources

- **Documentation**: https://docs.expo.dev/
- **Expo Router**: https://docs.expo.dev/router/introduction/
- **EAS**: https://docs.expo.dev/eas/
- **API Reference**: https://docs.expo.dev/versions/latest/
- **Community**: https://chat.expo.dev/
- **Forums**: https://forums.expo.dev/

## Summary

Expo streamlines React Native development:

1. **Managed workflow** eliminates native configuration
2. **Expo Go** for instant testing on real devices
3. **50+ built-in libraries** (camera, location, notifications)
4. **Expo Router** for file-based navigation
5. **EAS Build** for cloud builds
6. **OTA updates** for instant fixes
7. **TypeScript support** with full type safety

Perfect for rapid development, MVPs, and teams without native expertise. Upgrade to bare workflow or development builds when needing custom native code.
