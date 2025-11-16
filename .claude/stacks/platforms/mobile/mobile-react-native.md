# React Native Mobile Development Guide

## Overview

React Native is a framework for building native mobile applications using React and JavaScript/TypeScript. It allows you to create truly native apps for iOS and Android using a single codebase while maintaining native performance and platform-specific features.

**Key Features:**
- **Native Performance**: Real native components, not webviews
- **Single Codebase**: Write once, run on iOS and Android
- **Hot Reloading**: Instant feedback during development
- **Native APIs**: Access to camera, location, sensors, etc.
- **Large Ecosystem**: Extensive third-party libraries
- **Platform-Specific Code**: Customize per platform when needed
- **TypeScript Support**: Full type safety

**When to Use:**
- Building mobile apps for iOS and Android
- Need native performance and feel
- Want to share code between platforms
- Team has React/JavaScript experience
- Need access to native device features

## Installation

**📚 Official Documentation**: [React Native - Environment Setup](https://reactnative.dev/docs/set-up-your-environment)

Follow the official environment setup guide for detailed instructions for your OS (macOS, Windows, Linux) and target platforms (iOS, Android).

**Note**: React Native recommends using Expo for most new projects. See [mobile-expo.md](mobile-expo.md) for Expo setup.

### Quick Reference

**Prerequisites:**
- **macOS** (for iOS): Xcode, Command Line Tools, CocoaPods
- **Windows/Linux** (for Android): Android Studio, Java JDK, ANDROID_HOME

**Setup Process:**
1. Set up development environment (Xcode/Android Studio)
2. Create new project with React Native CLI
3. Install dependencies
4. Run on simulator/emulator or device

**Note**: Setup requirements and commands may change. Always refer to the official docs above for the current setup method.

**Project Structure:**
```
MyApp/
├── android/              # Android native code
├── ios/                  # iOS native code
├── src/
│   ├── components/       # React components
│   ├── screens/          # Screen components
│   ├── navigation/       # Navigation setup
│   ├── services/         # API calls, utilities
│   ├── hooks/            # Custom hooks
│   ├── store/            # State management
│   └── types/            # TypeScript types
├── App.tsx               # Root component
├── index.js              # Entry point
├── package.json
└── tsconfig.json
```

## Core Components

### View & Text

```typescript
import { View, Text, StyleSheet } from 'react-native';

export function WelcomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to React Native</Text>
      <Text style={styles.subtitle}>Build native mobile apps</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginTop: 8,
  },
});
```

### ScrollView & FlatList

**ScrollView (for small lists):**
```typescript
import { ScrollView, View, Text, StyleSheet } from 'react-native';

export function ProfileScreen() {
  return (
    <ScrollView style={styles.container}>
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>About</Text>
        <Text style={styles.sectionContent}>
          Lorem ipsum dolor sit amet...
        </Text>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Interests</Text>
        <Text style={styles.sectionContent}>
          Technology, Design, Travel
        </Text>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  section: {
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 8,
  },
  sectionContent: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
});
```

**FlatList (for large lists - virtualized):**
```typescript
import { FlatList, View, Text, StyleSheet, TouchableOpacity } from 'react-native';

interface User {
  id: string;
  name: string;
  email: string;
}

export function UserListScreen() {
  const [users, setUsers] = useState<User[]>([
    { id: '1', name: 'John Doe', email: 'john@example.com' },
    { id: '2', name: 'Jane Smith', email: 'jane@example.com' },
    // ... more users
  ]);

  const renderItem = ({ item }: { item: User }) => (
    <TouchableOpacity style={styles.userCard}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userEmail}>{item.email}</Text>
    </TouchableOpacity>
  );

  return (
    <FlatList
      data={users}
      renderItem={renderItem}
      keyExtractor={(item) => item.id}
      contentContainerStyle={styles.listContainer}
      ItemSeparatorComponent={() => <View style={styles.separator} />}
    />
  );
}

const styles = StyleSheet.create({
  listContainer: {
    padding: 16,
  },
  userCard: {
    padding: 16,
    backgroundColor: '#fff',
    borderRadius: 8,
  },
  userName: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
  },
  userEmail: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
  separator: {
    height: 12,
  },
});
```

### Buttons & Touchables

```typescript
import { TouchableOpacity, TouchableHighlight, Pressable, Text, StyleSheet } from 'react-native';

export function ButtonExamples() {
  return (
    <View style={styles.container}>
      {/* TouchableOpacity - fades on press */}
      <TouchableOpacity
        style={styles.button}
        activeOpacity={0.7}
        onPress={() => console.log('Pressed')}
      >
        <Text style={styles.buttonText}>TouchableOpacity</Text>
      </TouchableOpacity>

      {/* Pressable - modern, more control */}
      <Pressable
        style={({ pressed }) => [
          styles.button,
          pressed && styles.buttonPressed,
        ]}
        onPress={() => console.log('Pressed')}
      >
        {({ pressed }) => (
          <Text style={styles.buttonText}>
            {pressed ? 'Pressing...' : 'Pressable'}
          </Text>
        )}
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    gap: 12,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonPressed: {
    backgroundColor: '#0051D5',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

### TextInput

```typescript
import { useState } from 'react';
import { View, TextInput, Text, StyleSheet } from 'react-native';

export function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Email</Text>
      <TextInput
        style={styles.input}
        value={email}
        onChangeText={setEmail}
        placeholder="you@example.com"
        keyboardType="email-address"
        autoCapitalize="none"
        autoComplete="email"
      />

      <Text style={styles.label}>Password</Text>
      <TextInput
        style={styles.input}
        value={password}
        onChangeText={setPassword}
        placeholder="••••••••"
        secureTextEntry
        autoComplete="password"
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
    marginBottom: 8,
  },
  input: {
    height: 48,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 16,
    fontSize: 16,
    marginBottom: 16,
  },
});
```

## Navigation

### React Navigation Setup

**Installation:**
```bash
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context

# iOS only
cd ios && pod install && cd ..
```

**Basic Stack Navigator:**

**src/navigation/RootNavigator.tsx**
```typescript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { HomeScreen } from '../screens/HomeScreen';
import { ProfileScreen } from '../screens/ProfileScreen';
import { DetailsScreen } from '../screens/DetailsScreen';

export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Details: { itemId: string; title: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export function RootNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
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
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: 'Home' }}
        />
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{ title: 'Profile' }}
        />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={({ route }) => ({ title: route.params.title })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

**Type-Safe Navigation:**

**src/navigation/types.ts**
```typescript
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { RouteProp } from '@react-navigation/native';
import { RootStackParamList } from './RootNavigator';

export type HomeScreenNavigationProp = NativeStackNavigationProp<
  RootStackParamList,
  'Home'
>;

export type ProfileScreenRouteProp = RouteProp<
  RootStackParamList,
  'Profile'
>;

export type ProfileScreenNavigationProp = NativeStackNavigationProp<
  RootStackParamList,
  'Profile'
>;
```

**Using Navigation in Screens:**

**src/screens/HomeScreen.tsx**
```typescript
import { View, Text, Button, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { HomeScreenNavigationProp } from '../navigation/types';

export function HomeScreen() {
  const navigation = useNavigation<HomeScreenNavigationProp>();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Screen</Text>

      <Button
        title="Go to Profile"
        onPress={() => navigation.navigate('Profile', { userId: '123' })}
      />

      <Button
        title="Go to Details"
        onPress={() =>
          navigation.navigate('Details', {
            itemId: '456',
            title: 'Item Details',
          })
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    gap: 12,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
});
```

**src/screens/ProfileScreen.tsx**
```typescript
import { View, Text, StyleSheet } from 'react-native';
import { useRoute } from '@react-navigation/native';
import { ProfileScreenRouteProp } from '../navigation/types';

export function ProfileScreen() {
  const route = useRoute<ProfileScreenRouteProp>();
  const { userId } = route.params;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Profile Screen</Text>
      <Text>User ID: {userId}</Text>
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
    marginBottom: 12,
  },
});
```

### Tab Navigator

```typescript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { HomeScreen } from '../screens/HomeScreen';
import { SearchScreen } from '../screens/SearchScreen';
import { ProfileScreen } from '../screens/ProfileScreen';

const Tab = createBottomTabNavigator();

export function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>🏠</Text>
          ),
        }}
      />
      <Tab.Screen
        name="Search"
        component={SearchScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>🔍</Text>
          ),
        }}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Text style={{ fontSize: size, color }}>👤</Text>
          ),
        }}
      />
    </Tab.Navigator>
  );
}
```

## API Calls & Data Fetching

### Using Fetch

**src/services/api.ts**
```typescript
const API_BASE_URL = 'https://api.example.com';

export interface User {
  id: string;
  name: string;
  email: string;
}

export async function fetchUsers(): Promise<User[]> {
  try {
    const response = await fetch(`${API_BASE_URL}/users`);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
}

export async function createUser(userData: Omit<User, 'id'>): Promise<User> {
  const response = await fetch(`${API_BASE_URL}/users`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(userData),
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return response.json();
}
```

**Using in Component:**
```typescript
import { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator, StyleSheet } from 'react-native';
import { fetchUsers, User } from '../services/api';

export function UsersScreen() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    loadUsers();
  }, []);

  const loadUsers = async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await fetchUsers();
      setUsers(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>Error: {error}</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={users}
      renderItem={({ item }) => (
        <View style={styles.userItem}>
          <Text style={styles.userName}>{item.name}</Text>
          <Text style={styles.userEmail}>{item.email}</Text>
        </View>
      )}
      keyExtractor={(item) => item.id}
    />
  );
}

const styles = StyleSheet.create({
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  errorText: {
    color: 'red',
    fontSize: 16,
  },
  userItem: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  userName: {
    fontSize: 16,
    fontWeight: '600',
  },
  userEmail: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
});
```

## State Management

### Context API

**src/contexts/AuthContext.tsx**
```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const login = async (email: string, password: string) => {
    try {
      setIsLoading(true);
      // API call to login
      const response = await fetch('https://api.example.com/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      const data = await response.json();
      setUser(data.user);
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

**App.tsx**
```typescript
import { AuthProvider } from './src/contexts/AuthContext';
import { RootNavigator } from './src/navigation/RootNavigator';

export default function App() {
  return (
    <AuthProvider>
      <RootNavigator />
    </AuthProvider>
  );
}
```

## Native Features

### Camera

```bash
npm install react-native-image-picker
cd ios && pod install && cd ..
```

```typescript
import { useState } from 'react';
import { View, Image, Button, StyleSheet } from 'react-native';
import { launchCamera, launchImageLibrary } from 'react-native-image-picker';

export function CameraScreen() {
  const [imageUri, setImageUri] = useState<string | null>(null);

  const takePhoto = () => {
    launchCamera(
      {
        mediaType: 'photo',
        quality: 0.8,
      },
      (response) => {
        if (response.assets && response.assets[0]) {
          setImageUri(response.assets[0].uri || null);
        }
      }
    );
  };

  const pickImage = () => {
    launchImageLibrary(
      {
        mediaType: 'photo',
        quality: 0.8,
      },
      (response) => {
        if (response.assets && response.assets[0]) {
          setImageUri(response.assets[0].uri || null);
        }
      }
    );
  };

  return (
    <View style={styles.container}>
      {imageUri && (
        <Image source={{ uri: imageUri }} style={styles.image} />
      )}

      <Button title="Take Photo" onPress={takePhoto} />
      <Button title="Pick from Library" onPress={pickImage} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    gap: 12,
  },
  image: {
    width: '100%',
    height: 300,
    borderRadius: 8,
    marginBottom: 20,
  },
});
```

### Location

```bash
npm install @react-native-community/geolocation
cd ios && pod install && cd ..
```

```typescript
import { useState, useEffect } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import Geolocation from '@react-native-community/geolocation';

export function LocationScreen() {
  const [location, setLocation] = useState<{
    latitude: number;
    longitude: number;
  } | null>(null);

  const getCurrentLocation = () => {
    Geolocation.getCurrentPosition(
      (position) => {
        setLocation({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
        });
      },
      (error) => console.error(error),
      { enableHighAccuracy: true, timeout: 20000, maximumAge: 1000 }
    );
  };

  return (
    <View style={styles.container}>
      <Button title="Get Location" onPress={getCurrentLocation} />

      {location && (
        <View style={styles.locationInfo}>
          <Text>Latitude: {location.latitude}</Text>
          <Text>Longitude: {location.longitude}</Text>
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
  locationInfo: {
    marginTop: 20,
  },
});
```

## Platform-Specific Code

### Using Platform Module

```typescript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    marginTop: Platform.OS === 'ios' ? 20 : 0,
    padding: Platform.select({
      ios: 16,
      android: 12,
    }),
  },
});
```

### Separate Files

**Button.ios.tsx**
```typescript
export function Button() {
  return <iOSSpecificButton />;
}
```

**Button.android.tsx**
```typescript
export function Button() {
  return <AndroidSpecificButton />;
}
```

**Usage (automatically picks correct file):**
```typescript
import { Button } from './Button'; // Gets .ios or .android automatically
```

## Running the App

### iOS

```bash
# Run on simulator
npx react-native run-ios

# Run on specific device
npx react-native run-ios --device "My iPhone"

# Run specific simulator
npx react-native run-ios --simulator="iPhone 15 Pro"
```

### Android

```bash
# Start Metro bundler
npx react-native start

# Run on emulator/device (in new terminal)
npx react-native run-android

# Run on specific device
npx react-native run-android --deviceId=DEVICE_ID
```

## Best Practices

### 1. Use TypeScript

```typescript
// Good
interface UserProps {
  name: string;
  email: string;
  onPress: () => void;
}

export function UserCard({ name, email, onPress }: UserProps) {
  // Implementation
}

// Bad
export function UserCard({ name, email, onPress }) {
  // Implementation
}
```

### 2. Memoize Expensive Computations

```typescript
import { useMemo } from 'react';

function ExpensiveList({ data }) {
  const sortedData = useMemo(() => {
    return data.sort((a, b) => a.name.localeCompare(b.name));
  }, [data]);

  return <FlatList data={sortedData} />;
}
```

### 3. Use FlatList for Long Lists

```typescript
// Good: Virtualized rendering
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
/>

// Bad: Renders all items at once
<ScrollView>
  {items.map((item) => <Item key={item.id} {...item} />)}
</ScrollView>
```

### 4. Avoid Inline Functions in Render

```typescript
// Good
const handlePress = useCallback(() => {
  console.log('Pressed');
}, []);

<Button onPress={handlePress} />

// Bad: Creates new function every render
<Button onPress={() => console.log('Pressed')} />
```

### 5. Use StyleSheet.create

```typescript
// Good: Optimized by React Native
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});

// Bad: Created every render
<View style={{ flex: 1 }} />
```

## Environment Variables

```bash
npm install react-native-config
cd ios && pod install && cd ..
```

**.env**
```bash
API_URL=https://api.example.com
API_KEY=your_api_key_here
```

**Usage:**
```typescript
import Config from 'react-native-config';

const API_URL = Config.API_URL;
const API_KEY = Config.API_KEY;
```

## Debugging

### React Native Debugger

```bash
# Install
brew install --cask react-native-debugger

# Open debugger
open "rndebugger://set-debugger-loc?host=localhost&port=8081"
```

### Flipper

Built-in debugging tool for React Native:
- View logs
- Inspect network requests
- View React component tree
- Profile performance

## Resources

- **Documentation**: https://reactnative.dev/docs/getting-started
- **GitHub**: https://github.com/facebook/react-native
- **React Navigation**: https://reactnavigation.org/
- **Community**: https://reactnative.dev/community/overview
- **Awesome React Native**: https://github.com/jondot/awesome-react-native

## Summary

React Native enables native mobile development with React:

1. **Single codebase** for iOS and Android
2. **Native components** for real native performance
3. **Hot reloading** for fast development
4. **Large ecosystem** with extensive libraries
5. **Type-safe** with TypeScript support
6. **Platform-specific** code when needed
7. **Native APIs** for device features

Use React Navigation for routing, Context API or state management libraries for state, and platform-specific code when customization is needed per platform.
