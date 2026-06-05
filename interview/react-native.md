# React Native — Interview Questions (Junior → Senior)

## Table of Contents

1. [React Native vs React Web](#1-react-native-vs-react-web)
2. [Core Components](#2-core-components)
3. [FlatList Optimization](#3-flatlist-optimization)
4. [StyleSheet](#4-stylesheet)
5. [Flexbox trong React Native](#5-flexbox-trong-react-native)
6. [Navigation — React Navigation](#6-navigation--react-navigation)
7. [Platform-specific Code](#7-platform-specific-code)
8. [Bridging và Native Modules](#8-bridging-và-native-modules)
9. [Storage — AsyncStorage vs MMKV vs SecureStore](#9-storage--asyncstorage-vs-mmkv-vs-securestore)
10. [Performance Optimization](#10-performance-optimization)
11. [Push Notifications](#11-push-notifications)
12. [App Deployment — Expo vs Bare Workflow](#12-app-deployment--expo-vs-bare-workflow)
13. [New Architecture — JSI, Fabric, TurboModules](#13-new-architecture--jsi-fabric-turbomodules)
14. [Debugging](#14-debugging)
15. [Handling Keyboard](#15-handling-keyboard)

---

## 1. React Native vs React Web

**[Junior]**

> **EN:** What are the key differences between React Native and React for the web? How does rendering, styling, and navigation differ?
>
> **VI:** Những khác biệt chính giữa React Native và React Web là gì? Rendering, styling và navigation khác nhau như thế nào?

**Trả lời:**

React Native và React Web đều dùng React (component model, hooks, JSX) nhưng khác nhau hoàn toàn ở tầng render, styling, và các APIs nền tảng.

**Rendering:**

| | React Web | React Native |
|--|-----------|--------------|
| Output | HTML DOM (div, span, button) | Native views (UIView trên iOS, View trên Android) |
| Layout engine | CSS (browser engine) | Yoga layout engine (Flexbox subset) |
| Paint | Browser | Native renderer (Fabric trong New Architecture) |
| JavaScript bridge | Không cần | Cần (Old arch) / JSI (New arch) |

```typescript
// React Web
function Button() {
  return (
    <div className="btn" style={{ backgroundColor: 'blue' }}>
      <span>Click me</span>
    </div>
  );
}

// React Native — không có div, span, class
function Button() {
  return (
    <View style={{ backgroundColor: 'blue', padding: 12, borderRadius: 8 }}>
      <Text style={{ color: 'white' }}>Click me</Text>
    </View>
  );
}
```

**Styling:**

- React Web: CSS classes, CSS-in-JS, Tailwind — hỗ trợ đầy đủ CSS spec
- React Native: StyleSheet API — chỉ hỗ trợ subset của CSS, không có cascade, không có inheritance (ngoại trừ Text → Text)

**Navigation:**

- React Web: React Router, Next.js Router — URL-based, browser history API
- React Native: React Navigation (stack-based, không có URL trong app), Expo Router (file-based routing)

**APIs nền tảng:**

```typescript
// Web: window, document, localStorage, fetch
localStorage.setItem('key', 'value');

// React Native: không có DOM APIs, dùng React Native APIs
import AsyncStorage from '@react-native-async-storage/async-storage';
await AsyncStorage.setItem('key', 'value');

// Camera, GPS, Haptics — chỉ có trên native
import * as Haptics from 'expo-haptics';
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
```

---

## 2. Core Components

**[Junior]**

> **EN:** Walk me through the core React Native components and when to use each one.
>
> **VI:** Trình bày các core components của React Native và khi nào dùng từng cái?

**Trả lời:**

**View — container cơ bản:**

```typescript
import { View, StyleSheet } from 'react-native';

// View là "div" của React Native
// Hỗ trợ Flexbox, không render text trực tiếp
function Card({ children }) {
  return <View style={styles.card}>{children}</View>;
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 16,
    shadowColor: '#000',
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 2, // Android shadow
  },
});
```

**Text — bắt buộc cho mọi text:**

```typescript
import { Text } from 'react-native';

// Mọi text PHẢI nằm trong <Text>, không như HTML
// Text lồng nhau: style được kế thừa từ Text cha
function Typography() {
  return (
    <Text style={{ fontSize: 16, color: '#333' }}>
      Normal text{' '}
      <Text style={{ fontWeight: 'bold' }}>bold part</Text>
      {' '}back to normal
    </Text>
  );
}
```

**ScrollView vs FlatList:**

```typescript
// ScrollView — render TẤT CẢ children ngay lập tức
// Dùng cho: content có số lượng cố định, không quá nhiều
import { ScrollView } from 'react-native';

function ProfilePage() {
  return (
    <ScrollView
      showsVerticalScrollIndicator={false}
      contentContainerStyle={{ paddingBottom: 40 }}
    >
      <Header />
      <Bio />
      <Stats />
      {/* ... fixed number of sections */}
    </ScrollView>
  );
}

// FlatList — virtualized, chỉ render items trong viewport
// Dùng cho: danh sách dài, số lượng items không biết trước
import { FlatList } from 'react-native';

function ProductList({ products }) {
  return (
    <FlatList
      data={products}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ProductCard product={item} />}
      ItemSeparatorComponent={() => <View style={{ height: 12 }} />}
      ListEmptyComponent={<EmptyState />}
      ListHeaderComponent={<ListHeader />}
    />
  );
}

// SectionList — FlatList với header cho mỗi section
import { SectionList } from 'react-native';

function ContactList() {
  const sections = [
    { title: 'A', data: [{ name: 'Alice' }, { name: 'Anna' }] },
    { title: 'B', data: [{ name: 'Bob' }] },
  ];

  return (
    <SectionList
      sections={sections}
      keyExtractor={(item) => item.name}
      renderItem={({ item }) => <ContactRow contact={item} />}
      renderSectionHeader={({ section }) => (
        <Text style={styles.sectionHeader}>{section.title}</Text>
      )}
    />
  );
}
```

**TextInput:**

```typescript
import { TextInput } from 'react-native';

function SearchBar() {
  const [value, setValue] = useState('');
  const inputRef = useRef<TextInput>(null);

  return (
    <TextInput
      ref={inputRef}
      value={value}
      onChangeText={setValue}
      placeholder="Search..."
      placeholderTextColor="#999"
      autoCapitalize="none"
      autoCorrect={false}
      returnKeyType="search"
      onSubmitEditing={() => handleSearch(value)}
      // iOS: xóa nút X
      clearButtonMode="while-editing"
      // Keyboard types
      keyboardType="email-address" // hoặc "numeric", "phone-pad"
      secureTextEntry={true} // password
      style={styles.input}
    />
  );
}
```

**TouchableOpacity vs Pressable:**

```typescript
import { TouchableOpacity, Pressable } from 'react-native';

// TouchableOpacity — cũ hơn, đơn giản, vẫn phổ biến
function OldButton({ onPress, children }) {
  return (
    <TouchableOpacity onPress={onPress} activeOpacity={0.7}>
      {children}
    </TouchableOpacity>
  );
}

// Pressable — mới hơn (RN 0.63+), linh hoạt hơn
// Có thể style dựa theo pressed state
function ModernButton({ onPress, children }) {
  return (
    <Pressable
      onPress={onPress}
      onLongPress={() => console.log('long press')}
      style={({ pressed }) => ({
        opacity: pressed ? 0.7 : 1,
        backgroundColor: pressed ? '#0056b3' : '#007AFF',
        padding: 12,
        borderRadius: 8,
      })}
      // Khu vực hit lớn hơn visual area
      hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
    >
      {({ pressed }) => (
        <Text style={{ color: pressed ? '#eee' : 'white' }}>{children}</Text>
      )}
    </Pressable>
  );
}
```

---

## 3. FlatList Optimization

**[Mid → Senior]**

> **EN:** How do you optimize FlatList performance for long lists? Explain keyExtractor, getItemLayout, windowSize, maxToRenderPerBatch, removeClippedSubviews, and memo on renderItem.
>
> **VI:** Làm thế nào để tối ưu FlatList cho danh sách dài? Giải thích keyExtractor, getItemLayout, windowSize, maxToRenderPerBatch, removeClippedSubviews và memo renderItem?

**Trả lời:**

FlatList đã là virtualized list, nhưng với danh sách hàng nghìn items hoặc items phức tạp, cần tối ưu thêm.

```typescript
import React, { memo, useCallback } from 'react';
import { FlatList, View } from 'react-native';

// 1. Memo hóa renderItem component — tránh re-render không cần thiết
const ProductCard = memo(({ product, onPress }: ProductCardProps) => {
  return (
    <Pressable onPress={() => onPress(product.id)}>
      <Text>{product.name}</Text>
    </Pressable>
  );
});

const ITEM_HEIGHT = 80; // Chiều cao cố định của mỗi item

function OptimizedProductList({ products }) {
  // 2. useCallback cho renderItem — tránh tạo function mới mỗi lần render
  const renderItem = useCallback(
    ({ item }: { item: Product }) => (
      <ProductCard product={item} onPress={handlePress} />
    ),
    [handlePress] // dependencies
  );

  // 3. keyExtractor — string unique cho mỗi item, tránh dùng index
  const keyExtractor = useCallback((item: Product) => item.id, []);

  // 4. getItemLayout — chỉ dùng khi item có chiều cao CỐ ĐỊNH
  // Giúp FlatList tính offset mà không cần render item
  // → scrollToIndex, initialScrollIndex hoạt động chính xác
  const getItemLayout = useCallback(
    (_: any, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  return (
    <FlatList
      data={products}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}

      // 5. windowSize — số màn hình được render (trên + dưới viewport)
      // Default: 21 (10 màn hình trên + viewport + 10 màn hình dưới)
      // Giảm xuống để tiết kiệm memory, nhưng tăng blank areas khi scroll nhanh
      windowSize={10}

      // 6. maxToRenderPerBatch — số items render mỗi batch khi scroll
      // Default: 10. Giảm → scroll smoother nhưng blank areas nhiều hơn
      maxToRenderPerBatch={5}

      // 7. initialNumToRender — số items render lần đầu
      // Nên đủ để fill màn hình
      initialNumToRender={15}

      // 8. removeClippedSubviews — unmount items ngoài viewport
      // iOS: thường không cần (native handles it)
      // Android: giúp nhiều với danh sách rất dài
      removeClippedSubviews={true}

      // 9. updateCellsBatchingPeriod — delay giữa các batch (ms)
      // Default: 50ms
      updateCellsBatchingPeriod={100}

      // Separator
      ItemSeparatorComponent={() => (
        <View style={{ height: 1, backgroundColor: '#eee' }} />
      )}

      // Tắt scroll indicator cho UI gọn hơn
      showsVerticalScrollIndicator={false}
    />
  );
}
```

**Tối ưu nâng cao:**

```typescript
// Sử dụng FlashList (Shopify) thay FlatList
// FlashList nhanh hơn ~10x nhờ recycling views thay vì unmount
import { FlashList } from '@shopify/flash-list';

function FastList({ products }) {
  return (
    <FlashList
      data={products}
      renderItem={({ item }) => <ProductCard product={item} />}
      estimatedItemSize={80} // ước tính height để tính layout
      keyExtractor={(item) => item.id}
    />
  );
}

// Tránh anonymous functions trong renderItem
// Sai:
<FlatList renderItem={({ item }) => <Card item={item} onPress={() => navigate(item.id)} />} />

// Đúng:
const handlePress = useCallback((id: string) => navigate(id), [navigate]);
const renderItem = useCallback(({ item }) => (
  <Card item={item} onPress={handlePress} />
), [handlePress]);
```

---

## 4. StyleSheet

**[Junior]**

> **EN:** What is StyleSheet.create? Why use it instead of inline styles? What are the performance implications?
>
> **VI:** StyleSheet.create là gì? Tại sao dùng thay vì inline styles? Ảnh hưởng đến performance như thế nào?

**Trả lời:**

**StyleSheet.create vs inline styles:**

```typescript
import { StyleSheet, View, Text } from 'react-native';

// Inline styles — tạo object mới mỗi lần render → nhiều GC pressure
function BadComponent() {
  return (
    <View style={{ backgroundColor: 'blue', padding: 16, borderRadius: 8 }}>
      <Text style={{ color: 'white', fontSize: 16 }}>Hello</Text>
    </View>
  );
}

// StyleSheet.create — tạo một lần, tối ưu hơn
const styles = StyleSheet.create({
  container: {
    backgroundColor: 'blue',
    padding: 16,
    borderRadius: 8,
  },
  text: {
    color: 'white',
    fontSize: 16,
  },
});

function GoodComponent() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello</Text>
    </View>
  );
}
```

**Lợi ích của StyleSheet.create:**

1. **Validation**: Báo lỗi nếu dùng property không hợp lệ trong development
2. **Performance**: Styles được gửi qua bridge một lần và tham chiếu bằng ID số nguyên
3. **Readability**: Tách styles ra khỏi JSX, dễ đọc hơn
4. **Type safety**: TypeScript autocomplete và type checking

**Dynamic styles — kết hợp static và dynamic:**

```typescript
const styles = StyleSheet.create({
  button: {
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  text: {
    fontWeight: '600',
  },
});

function Button({ variant = 'primary', disabled = false }) {
  return (
    <Pressable
      style={[
        styles.button,
        // Dynamic styles dựa theo props
        { backgroundColor: variant === 'primary' ? '#007AFF' : '#666' },
        disabled && { opacity: 0.5 },
      ]}
      disabled={disabled}
    >
      <Text style={[styles.text, { color: 'white' }]}>Click</Text>
    </Pressable>
  );
}
```

**StyleSheet utilities:**

```typescript
// StyleSheet.flatten — merge và flatten style array thành plain object
const merged = StyleSheet.flatten([styles.base, styles.override, { color: 'red' }]);

// StyleSheet.absoluteFill — shortcut cho position: 'absolute', top/left/right/bottom: 0
<View style={StyleSheet.absoluteFill} />
// Tương đương:
<View style={{ position: 'absolute', top: 0, left: 0, right: 0, bottom: 0 }} />
```

---

## 5. Flexbox trong React Native

**[Junior]**

> **EN:** How does Flexbox in React Native differ from CSS Flexbox on the web?
>
> **VI:** Flexbox trong React Native khác Flexbox CSS trên web như thế nào?

**Trả lời:**

React Native dùng Yoga layout engine của Meta — là implementation của Flexbox nhưng có nhiều khác biệt so với CSS web.

**Khác biệt quan trọng:**

| | CSS Web | React Native |
|--|---------|--------------|
| `flexDirection` default | `row` | **`column`** |
| `alignContent` default | `stretch` | `flex-start` |
| `flexShrink` default | `1` | **`0`** |
| `position` | relative/absolute/fixed/sticky | relative/absolute (không có fixed/sticky) |
| Grid layout | Có (`display: grid`) | **Không có** |
| `%` units | Hầu hết properties | Chỉ một số (width, height, padding, margin) |
| Cascade | Có | Không (ngoại trừ Text → Text) |

```typescript
import { View, Text, StyleSheet } from 'react-native';

// Default: flexDirection column (khác web!)
function Column() {
  return (
    <View style={{ flex: 1 }}>
      {/* Các children xếp từ trên xuống */}
      <View style={{ height: 50, backgroundColor: 'red' }} />
      <View style={{ height: 50, backgroundColor: 'blue' }} />
    </View>
  );
}

// Row layout
function Row() {
  return (
    <View style={{ flexDirection: 'row', alignItems: 'center' }}>
      <Text>Label</Text>
      <View style={{ flex: 1 }} /> {/* Spacer */}
      <Text>Value</Text>
    </View>
  );
}

// Centering — cực kỳ phổ biến
function Centered({ children }) {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      {children}
    </View>
  );
}

// Responsive layout — không có media queries
import { Dimensions, useWindowDimensions } from 'react-native';

function ResponsiveGrid() {
  const { width } = useWindowDimensions(); // re-renders khi rotate
  const numColumns = width > 600 ? 3 : 2;
  const itemWidth = (width - 32 - (numColumns - 1) * 12) / numColumns;

  return (
    <FlatList
      numColumns={numColumns}
      key={numColumns} // key thay đổi → FlatList remount khi numColumns thay đổi
      data={items}
      renderItem={({ item }) => (
        <View style={{ width: itemWidth }}>
          <ProductCard product={item} />
        </View>
      )}
    />
  );
}
```

**SafeAreaView — xử lý notch và home indicator:**

```typescript
import { SafeAreaView } from 'react-native-safe-area-context';

function Screen() {
  return (
    // Tự động tránh notch, status bar, home indicator
    <SafeAreaView style={{ flex: 1 }} edges={['top', 'left', 'right']}>
      <Content />
    </SafeAreaView>
  );
}
```

---

## 6. Navigation — React Navigation

**[Mid]**

> **EN:** Explain React Navigation. How do you set up Stack, Tab, and Drawer navigators? How do you pass params and implement deep linking?
>
> **VI:** Giải thích React Navigation. Cách cấu hình Stack, Tab, Drawer? Truyền params và deep linking như thế nào?

**Trả lời:**

React Navigation là thư viện navigation chuẩn cho React Native. Dùng JavaScript navigation (khác với React Native Navigation dùng native navigation).

**Cấu trúc cơ bản:**

```typescript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

// Type-safe navigation với TypeScript
type RootStackParamList = {
  Home: undefined;
  ProductDetail: { productId: string; productName: string };
  Cart: undefined;
};

type TabParamList = {
  HomeTab: undefined;
  SearchTab: undefined;
  ProfileTab: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();
const Tab = createBottomTabNavigator<TabParamList>();

function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          const icons: Record<string, string> = {
            HomeTab: 'home',
            SearchTab: 'search',
            ProfileTab: 'person',
          };
          return <Icon name={icons[route.name]} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
      })}
    >
      <Tab.Screen name="HomeTab" component={HomeScreen} />
      <Tab.Screen name="SearchTab" component={SearchScreen} />
      <Tab.Screen name="ProfileTab" component={ProfileScreen} />
    </Tab.Navigator>
  );
}

function RootNavigator() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={TabNavigator} options={{ headerShown: false }} />
      <Stack.Screen
        name="ProductDetail"
        component={ProductDetailScreen}
        options={({ route }) => ({ title: route.params.productName })}
      />
    </Stack.Navigator>
  );
}

function App() {
  return (
    <NavigationContainer linking={linkingConfig}>
      <RootNavigator />
    </NavigationContainer>
  );
}
```

**Truyền params và type-safe navigation:**

```typescript
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { useNavigation } from '@react-navigation/native';

// Trong màn hình nhận params
type ProductDetailProps = NativeStackScreenProps<RootStackParamList, 'ProductDetail'>;

function ProductDetailScreen({ route, navigation }: ProductDetailProps) {
  const { productId, productName } = route.params;

  return (
    <View>
      <Text>{productName}</Text>
      <Button onPress={() => navigation.goBack()} title="Back" />
      <Button
        onPress={() => navigation.navigate('Cart')}
        title="Go to Cart"
      />
    </View>
  );
}

// Trong màn hình navigate đến
function HomeScreen() {
  const navigation = useNavigation<NativeStackNavigationProp<RootStackParamList>>();

  return (
    <Button
      onPress={() =>
        navigation.navigate('ProductDetail', {
          productId: '123',
          productName: 'iPhone 15',
        })
      }
      title="View Product"
    />
  );
}
```

**Navigation Ref — navigate từ ngoài component:**

```typescript
// Hữu ích khi navigate từ Redux action, notification handler, etc.
import { createNavigationContainerRef } from '@react-navigation/native';

export const navigationRef = createNavigationContainerRef<RootStackParamList>();

// Trong App.tsx
<NavigationContainer ref={navigationRef}>

// Bất kỳ nơi nào trong app
import { navigationRef } from './navigation/ref';

function handlePushNotification(notification) {
  if (navigationRef.isReady()) {
    navigationRef.navigate('ProductDetail', { productId: notification.data.id });
  }
}
```

**Deep Linking:**

```typescript
const linkingConfig = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: '',
      ProductDetail: 'product/:productId',
      // myapp://product/123 → ProductDetailScreen với productId="123"
      Cart: 'cart',
    },
  },
};
```

---

## 7. Platform-specific Code

**[Mid]**

> **EN:** How do you write platform-specific code in React Native? Explain Platform.OS, Platform.select, and platform-specific file extensions.
>
> **VI:** Cách viết code khác nhau cho từng platform? Platform.OS, Platform.select và file extension theo platform?

**Trả lời:**

React Native cung cấp nhiều cách để viết code khác nhau cho iOS và Android.

**Platform.OS và Platform.select:**

```typescript
import { Platform, StyleSheet } from 'react-native';

// Platform.OS — string 'ios' | 'android' | 'web' | 'windows' | 'macos'
function Header() {
  return (
    <View style={styles.header}>
      {/* Chỉ show back button trên Android (iOS có swipe back) */}
      {Platform.OS === 'android' && (
        <BackButton onPress={() => navigation.goBack()} />
      )}
      <Title />
    </View>
  );
}

// Platform.select — cleaner cho nhiều cases
const styles = StyleSheet.create({
  shadow: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 4,
    },
    android: {
      elevation: 4,
    },
    default: {}, // web hoặc platform khác
  }),

  // Có thể dùng trong value trực tiếp
  headerPaddingTop: {
    paddingTop: Platform.select({ ios: 44, android: 24 }),
  },
});

// Platform.Version — version của OS
if (Platform.OS === 'ios' && parseInt(Platform.Version as string, 10) >= 16) {
  // iOS 16+ specific feature
}

if (Platform.OS === 'android' && Platform.Version >= 31) {
  // Android 12+ specific
}
```

**Platform-specific file extensions:**

```
// React Native sẽ tự động pick đúng file:
Button.ios.tsx      ← dùng cho iOS
Button.android.tsx  ← dùng cho Android
Button.tsx          ← fallback nếu không có file platform-specific

// Cách dùng — import bình thường, không cần extension
import Button from './Button';
// React Native bundler (Metro) tự động resolve
```

```typescript
// Button.ios.tsx
import { TouchableHighlight } from 'react-native';

export default function Button({ onPress, title }) {
  return (
    <TouchableHighlight
      onPress={onPress}
      underlayColor="#ddd"
      style={styles.iosButton}
    >
      <Text>{title}</Text>
    </TouchableHighlight>
  );
}

// Button.android.tsx
import { TouchableNativeFeedback } from 'react-native';

export default function Button({ onPress, title }) {
  return (
    <TouchableNativeFeedback
      onPress={onPress}
      background={TouchableNativeFeedback.Ripple('#ddd', false)}
    >
      <View style={styles.androidButton}>
        <Text>{title}</Text>
      </View>
    </TouchableNativeFeedback>
  );
}
```

**Xử lý status bar:**

```typescript
import { StatusBar } from 'expo-status-bar';

function Screen() {
  return (
    <View>
      {/* iOS: light/dark content; Android: translucent, backgroundColor */}
      <StatusBar style="dark" backgroundColor="white" translucent={false} />
      <Content />
    </View>
  );
}
```

---

## 8. Bridging và Native Modules

**[Senior]**

> **EN:** When do you need native modules? How does bridging work? How does the New Architecture (JSI) change this?
>
> **VI:** Khi nào cần native modules? Bridge hoạt động như thế nào? New Architecture (JSI) thay đổi điều gì?

**Trả lời:**

**Khi nào cần Native Modules:**
- Truy cập API của platform mà React Native chưa có wrapper
- Tích hợp native SDK (Stripe, Google Maps, Face ID)
- Performance-critical operations (image processing, crypto)
- Tái dùng native code từ iOS/Android codebase hiện có

**Old Architecture — JavaScript Bridge:**

```
JavaScript Thread → [Bridge: JSON serialization] → Native Thread
                  ← [Bridge: JSON deserialization] ←
```

Vấn đề với Old Bridge:
- **Async only**: Mọi giao tiếp đều bất đồng bộ, không thể gọi synchronously
- **JSON overhead**: Dữ liệu phải serialize/deserialize qua JSON
- **Batched**: Messages được gom lại, gây delay

**New Architecture — JSI (JavaScript Interface):**

```
JavaScript Thread → [JSI: Direct C++ reference] → Native Thread
                    (no serialization needed)
```

```typescript
// Old Architecture Native Module (React Native < 0.68)
// NativeModules là bridge qua JSON
import { NativeModules } from 'react-native';

const { BiometricModule } = NativeModules;

// Phải async vì đi qua bridge
BiometricModule.authenticate(
  'Confirm your identity',
  (success) => console.log('Auth success'),
  (error) => console.log('Auth failed:', error)
);

// New Architecture — TurboModule
// Typed interface được generate từ TypeScript spec
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  authenticate(reason: string): Promise<boolean>;
  isAvailable(): boolean; // Có thể synchronous với JSI!
}

export default TurboModuleRegistry.getEnforcing<Spec>('BiometricModule');
```

**Tạo Native Module với Expo Modules API (recommended):**

```typescript
// expo-modules-core — cách đơn giản nhất để tạo native module
// expo/my-module/src/MyModuleView.tsx

import { requireNativeViewManager } from 'expo-modules-core';
import * as React from 'react';

const NativeView = requireNativeViewManager('MyModule');

export function MyModuleView(props: ViewProps) {
  return <NativeView {...props} />;
}
```

---

## 9. Storage — AsyncStorage vs MMKV vs SecureStore

**[Mid]**

> **EN:** Compare AsyncStorage, MMKV, and SecureStore. When do you use each?
>
> **VI:** So sánh AsyncStorage, MMKV và SecureStore. Khi nào dùng từng cái?

**Trả lời:**

| | AsyncStorage | MMKV | SecureStore |
|--|--------------|------|-------------|
| **Speed** | Chậm (async, disk I/O) | Rất nhanh (memory-mapped) | Chậm vừa |
| **Sync read** | Không | Có | Không |
| **Encryption** | Không | Optional | Có (Keychain/Keystore) |
| **Size limit** | 6MB (Android) | Không giới hạn | ~2KB/item |
| **Dùng khi** | Data đơn giản, không nhạy cảm | Cache lớn, cần tốc độ | Token, mật khẩu |

**AsyncStorage:**

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Chỉ lưu được string — phải JSON.stringify/parse
const storeUser = async (user: User) => {
  try {
    await AsyncStorage.setItem('user', JSON.stringify(user));
  } catch (error) {
    console.error('Failed to save user:', error);
  }
};

const getUser = async (): Promise<User | null> => {
  try {
    const json = await AsyncStorage.getItem('user');
    return json ? JSON.parse(json) : null;
  } catch {
    return null;
  }
};

// Multi operations — batch
const multiGet = async () => {
  const pairs = await AsyncStorage.multiGet(['user', 'settings', 'cart']);
  return Object.fromEntries(pairs.map(([k, v]) => [k, v ? JSON.parse(v) : null]));
};
```

**MMKV (react-native-mmkv):**

```typescript
import { MMKV } from 'react-native-mmkv';

// Instance dùng cho toàn app
export const storage = new MMKV();

// Encrypted instance
export const secureStorage = new MMKV({
  id: 'secure-storage',
  encryptionKey: 'my-encryption-key', // Nên lấy từ secure source
});

// Synchronous read/write — điểm mạnh nhất của MMKV
storage.set('user.name', 'Alice');
const name = storage.getString('user.name'); // sync!

storage.set('count', 42);
const count = storage.getNumber('count');

storage.set('isLoggedIn', true);
const loggedIn = storage.getBoolean('isLoggedIn');

// Dùng với Zustand persist
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();
const zustandStorage = {
  getItem: (name) => storage.getString(name) ?? null,
  setItem: (name, value) => storage.set(name, value),
  removeItem: (name) => storage.delete(name),
};

const useStore = create()(
  persist(
    (set) => ({ count: 0, increment: () => set((s) => ({ count: s.count + 1 })) }),
    { name: 'app-store', storage: createJSONStorage(() => zustandStorage) }
  )
);
```

**SecureStore (Expo):**

```typescript
import * as SecureStore from 'expo-secure-store';

// Lưu auth token — dùng iOS Keychain, Android Keystore
const saveAuthToken = async (token: string) => {
  await SecureStore.setItemAsync('auth_token', token, {
    keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
  });
};

const getAuthToken = async (): Promise<string | null> => {
  return SecureStore.getItemAsync('auth_token');
};

const deleteAuthToken = async () => {
  await SecureStore.deleteItemAsync('auth_token');
};
```

---

## 10. Performance Optimization

**[Senior]**

> **EN:** How do you optimize React Native app performance? Explain InteractionManager, useMemo, FlatList tuning, the Hermes engine, and Reanimated vs Animated.
>
> **VI:** Tối ưu performance React Native như thế nào? Giải thích InteractionManager, useMemo, FlatList tuning, Hermes engine và Reanimated vs Animated?

**Trả lời:**

**Hermes Engine:**

Hermes là JavaScript engine được tối ưu cho React Native (default từ RN 0.70+). Lợi ích:
- **Pre-compilation**: JS được compile thành bytecode lúc build, giảm startup time
- **Smaller memory footprint**: Dùng ít RAM hơn V8/JSC
- **Faster TTI**: Time to Interactive giảm đáng kể

```typescript
// Kiểm tra có đang dùng Hermes không
const isHermes = () => !!global.HermesInternal;
console.log('Using Hermes:', isHermes());
```

**InteractionManager — defer heavy work:**

```typescript
import { InteractionManager } from 'react-native';

function HeavyScreen() {
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    // Đợi navigation animation hoàn thành TRƯỚC KHI render heavy content
    // Tránh janky animation khi navigate đến màn hình phức tạp
    const task = InteractionManager.runAfterInteractions(() => {
      setIsReady(true);
    });

    return () => task.cancel();
  }, []);

  if (!isReady) {
    return <LoadingPlaceholder />;
  }

  return <HeavyContent />;
}
```

**Reanimated vs Animated:**

```typescript
// Animated (built-in) — chạy trên JS thread
// Vấn đề: animation bị ảnh hưởng nếu JS thread bận
import { Animated } from 'react-native';

function OldAnimation() {
  const opacity = useRef(new Animated.Value(0)).current;

  const fadeIn = () => {
    Animated.timing(opacity, {
      toValue: 1,
      duration: 300,
      useNativeDriver: true, // Chạy trên UI thread — tốt hơn nhưng giới hạn
    }).start();
  };

  return <Animated.View style={{ opacity }} />;
}

// Reanimated 3 — chạy trên UI thread (worklets)
// Không bị block bởi JS thread → animation luôn 60/120fps
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
  runOnJS,
} from 'react-native-reanimated';

function NewAnimation() {
  const opacity = useSharedValue(0);
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ scale: scale.value }],
  }));

  const fadeIn = () => {
    opacity.value = withTiming(1, { duration: 300 });
    scale.value = withSpring(1.05, {}, () => {
      scale.value = withSpring(1);
      // Gọi JS function từ worklet
      runOnJS(console.log)('Animation done');
    });
  };

  return (
    <Animated.View style={[styles.box, animatedStyle]}>
      <Pressable onPress={fadeIn}>
        <Text>Animate</Text>
      </Pressable>
    </Animated.View>
  );
}
```

**useMemo và useCallback — tránh re-render:**

```typescript
function ProductScreen({ category, userId }) {
  // Tính toán phức tạp — chỉ tính lại khi products hoặc filters thay đổi
  const filteredProducts = useMemo(
    () => products.filter(p => p.category === category && p.inStock),
    [products, category]
  );

  // Stable reference cho callback — tránh child re-render
  const handleAddToCart = useCallback(
    (product: Product) => {
      cartStore.addItem(product);
      analytics.track('add_to_cart', { userId, productId: product.id });
    },
    [userId] // cartStore stable, analytics stable
  );

  return <ProductList products={filteredProducts} onAdd={handleAddToCart} />;
}
```

**RAM Bundle / Inline Requires:**

```typescript
// metro.config.js — lazy loading modules
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        inlineRequires: true, // Load modules khi cần, không phải lúc startup
      },
    }),
  },
};
```

---

## 11. Push Notifications

**[Mid]**

> **EN:** How do push notifications work in React Native? Compare Expo Notifications and Firebase FCM. How do you handle foreground vs background notifications?
>
> **VI:** Push notifications hoạt động như thế nào trong React Native? So sánh Expo Notifications và Firebase FCM. Xử lý foreground vs background notifications?

**Trả lời:**

**Cách push notifications hoạt động:**

```
Backend → FCM/APNs → Device OS → App
                                  ├── Foreground: app đang mở → onMessage handler
                                  ├── Background: app đang chạy nhưng không focus → system tray
                                  └── Killed: app không chạy → system tray → user tap → app opens
```

**Expo Notifications (Expo Managed/Bare):**

```typescript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

// Cấu hình hiển thị khi foreground
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

async function registerForPushNotifications(): Promise<string | null> {
  if (!Device.isDevice) {
    console.warn('Must use physical device for push notifications');
    return null;
  }

  // Xin quyền
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    return null;
  }

  // Android channel (bắt buộc cho Android 8+)
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'Default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
    });
  }

  // Lấy Expo Push Token
  const token = await Notifications.getExpoPushTokenAsync({
    projectId: Constants.expoConfig?.extra?.eas?.projectId,
  });

  return token.data; // Gửi token này lên backend
}

// Lắng nghe notifications trong component
function App() {
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();

  useEffect(() => {
    registerForPushNotifications().then(token => {
      if (token) api.savePushToken(token);
    });

    // Foreground notification handler
    notificationListener.current = Notifications.addNotificationReceivedListener(
      (notification) => {
        console.log('Foreground notification:', notification);
        // Có thể update badge, show in-app toast, etc.
      }
    );

    // User tapped notification (từ background hoặc killed state)
    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        const { data } = response.notification.request.content;
        // Navigate đến màn hình phù hợp
        if (data.type === 'order') {
          navigationRef.navigate('OrderDetail', { orderId: data.orderId });
        }
      }
    );

    return () => {
      notificationListener.current?.remove();
      responseListener.current?.remove();
    };
  }, []);
}
```

**Firebase FCM (React Native Firebase):**

```typescript
import messaging from '@react-native-firebase/messaging';

// Foreground handler
const unsubscribeForeground = messaging().onMessage(async (remoteMessage) => {
  console.log('FCM foreground message:', remoteMessage);
  // Expo Notifications không show tự động khi foreground với FCM
  // Phải show thủ công
  await Notifications.scheduleNotificationAsync({
    content: {
      title: remoteMessage.notification?.title,
      body: remoteMessage.notification?.body,
      data: remoteMessage.data,
    },
    trigger: null,
  });
});

// Background handler — phải đăng ký bên ngoài component (module level)
messaging().setBackgroundMessageHandler(async (remoteMessage) => {
  console.log('FCM background message:', remoteMessage);
  // Không thể update UI, chỉ làm background task
});

// Notification tapped khi app ở background
messaging().onNotificationOpenedApp((remoteMessage) => {
  handleNotificationNavigation(remoteMessage.data);
});

// Notification tapped khi app bị killed
messaging()
  .getInitialNotification()
  .then((remoteMessage) => {
    if (remoteMessage) {
      handleNotificationNavigation(remoteMessage.data);
    }
  });
```

---

## 12. App Deployment — Expo vs Bare Workflow

**[Mid → Senior]**

> **EN:** Compare Expo Managed Workflow and bare workflow. What is EAS Build? How do you release to the App Store and Google Play?
>
> **VI:** So sánh Expo Managed và bare workflow. EAS Build là gì? Quy trình release lên App Store và Google Play?

**Trả lời:**

**Expo Managed vs Bare Workflow:**

| | Expo Managed | Bare Workflow |
|--|--------------|---------------|
| Native code | Không có (Expo quản lý) | Có android/ và ios/ |
| Flexibility | Giới hạn trong Expo APIs | Full native access |
| Setup | Nhanh | Phức tạp hơn |
| Native modules | Chỉ Expo SDK | Bất kỳ native module |
| Config plugins | Có | Có nhưng có thể edit trực tiếp |
| OTA updates | Có (expo-updates) | Có (với expo-updates) |
| Khi nào dùng | Prototype, app chuẩn | Cần custom native code |

**EAS Build — Cloud build service:**

```bash
# Cài EAS CLI
npm install -g eas-cli

# Login và cấu hình
eas login
eas build:configure

# eas.json — cấu hình build profiles
```

```json
{
  "cli": { "version": ">= 5.0.0" },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": { "simulator": true }
    },
    "preview": {
      "distribution": "internal",
      "android": { "buildType": "apk" }
    },
    "production": {
      "autoIncrement": true,
      "ios": { "resourceClass": "m-medium" }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "dev@company.com",
        "ascAppId": "1234567890"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production"
      }
    }
  }
}
```

```bash
# Build
eas build --platform ios --profile production
eas build --platform android --profile production
eas build --platform all --profile production

# Submit lên stores
eas submit --platform ios
eas submit --platform android

# OTA update (không cần qua App Store review)
eas update --branch production --message "Fix critical bug"
```

**Quy trình release thực tế:**

```
1. Tăng version trong app.json / package.json
2. EAS Build: eas build --platform all --profile production
3. Test build trên TestFlight (iOS) và Internal Testing (Android)
4. EAS Submit: eas submit --platform all
5. App Store Connect: điền metadata, screenshots, submit for review (1-3 ngày)
6. Google Play Console: submit, thường review < 24h
```

**App versioning:**

```json
// app.json
{
  "expo": {
    "version": "1.2.3",      // Hiển thị với người dùng
    "ios": {
      "buildNumber": "45"    // Tăng mỗi lần submit (CFBundleVersion)
    },
    "android": {
      "versionCode": 45      // Tăng mỗi lần submit (versionCode)
    }
  }
}
```

---

## 13. New Architecture — JSI, Fabric, TurboModules

**[Senior]**

> **EN:** What changed in React Native's New Architecture? Explain JSI, Fabric renderer, and TurboModules.
>
> **VI:** React Native New Architecture thay đổi gì? Giải thích JSI, Fabric renderer và TurboModules?

**Trả lời:**

New Architecture (default từ RN 0.74) giải quyết những hạn chế cơ bản của Old Bridge.

**Old Architecture — vấn đề:**

```
JavaScript Thread
     ↕ [Async Bridge: JSON serialization]
Native Thread

Vấn đề:
- Tất cả communication là async → không thể synchronous native calls
- JSON serialization tốn CPU, đặc biệt với data lớn
- Mọi thứ phải qua bridge → bottleneck
- Layout calculation chỉ xảy ra sau khi JS gửi instructions qua bridge
```

**New Architecture — giải pháp:**

```
JavaScript (Hermes) ← JSI → C++ Host Objects → iOS/Android Native

JSI (JavaScript Interface):
- C++ layer cho phép JS giữ reference trực tiếp đến native objects
- Không cần serialize/deserialize
- Có thể gọi synchronously
- Shared memory giữa JS và native
```

**TurboModules — lazy loading native modules:**

```typescript
// Old: tất cả NativeModules load khi startup
import { NativeModules } from 'react-native';
const { Camera } = NativeModules; // loaded ngay cả khi không dùng

// New: TurboModules — chỉ load khi cần
// Native module được load lazily, tiết kiệm startup time

// TypeScript spec (được dùng để generate code)
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  // Synchronous method — có thể nhờ JSI!
  getCurrentLocation(): {latitude: number; longitude: number};
  // Async method
  requestPermission(): Promise<boolean>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('LocationModule');
```

**Fabric — New Renderer:**

```
Old: Shadow Thread → [bridge] → Main Thread (UI updates)
New: C++ Shadow Tree → direct → Main Thread (no bridge)

Fabric lợi ích:
- Concurrent features: React 18 concurrent mode support
- Synchronous layout: host components có thể đọc layout synchronously
- Renderer chia sẻ logic giữa iOS/Android/Windows/macOS
```

**Concurrent Mode compatibility:**

```typescript
// Với New Architecture, React 18 concurrent features hoạt động đầy đủ
import { startTransition } from 'react';

function SearchScreen() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleSearch = (text: string) => {
    setQuery(text); // urgent update — UI update ngay
    startTransition(() => {
      setResults(filterProducts(text)); // non-urgent — có thể defer
    });
  };
}
```

---

## 14. Debugging

**[Mid]**

> **EN:** What tools do you use to debug React Native apps? Explain Flipper, React Native Debugger, performance profiler, and network inspector.
>
> **VI:** Bạn dùng tools nào để debug React Native? Giải thích Flipper, React Native Debugger, performance profiler và network inspector?

**Trả lời:**

**React Native built-in debugging:**

```typescript
// Dev Menu (shake device hoặc Cmd+D trên simulator)
// → Reload, Debug, Toggle Inspector, Performance Monitor

// Console
console.log('Debug:', value);
console.warn('Warning:', obj); // Hiện yellow box
console.error('Error:', err);  // Hiện red box

// __DEV__ — chỉ chạy trong development
if (__DEV__) {
  console.log('Development only log');
}
```

**React DevTools (Standalone):**

```bash
# Inspect component tree, props, state — như trên web
npx react-devtools
```

**Flipper (Meta's debugging platform):**

```
Flipper plugins hữu ích:
- React DevTools: Component tree, props, state
- Network Inspector: HTTP requests/responses
- Layout Inspector: View hierarchy, touch areas
- Hermes Debugger: Breakpoints, call stack, memory
- Logs: Device logs filter
- Crash Reporter: Crash logs từ production
```

**Logging với Reactotron:**

```typescript
// reactotron.config.ts — setup một lần
import Reactotron from 'reactotron-react-native';

if (__DEV__) {
  Reactotron
    .configure({ name: 'MyApp' })
    .useReactNative()
    .connect();

  // Override console để log vào Reactotron
  console.tron = Reactotron;
}

// Trong app
console.tron?.log('Custom log', { data: someObject });
console.tron?.display({
  name: 'API Call',
  value: response,
  preview: 'GET /products',
});
```

**Network debugging:**

```typescript
// Bắt network requests không qua Flipper
// Thêm vào index.js hoặc App.tsx
if (__DEV__) {
  global.XMLHttpRequest = global.originalXMLHttpRequest ?? global.XMLHttpRequest;
  global.FormData = global.originalFormData ?? global.FormData;
}

// Hoặc dùng Axios interceptors để log
axios.interceptors.request.use((config) => {
  console.log(`→ ${config.method?.toUpperCase()} ${config.url}`);
  return config;
});

axios.interceptors.response.use(
  (response) => {
    console.log(`← ${response.status} ${response.config.url}`);
    return response;
  },
  (error) => {
    console.error(`✗ ${error.response?.status} ${error.config.url}`);
    return Promise.reject(error);
  }
);
```

**Performance Profiling:**

```typescript
// Perf Monitor (Dev Menu → Show Perf Monitor)
// Hiện: FPS (JS thread và UI thread), RAM usage

// Systrace — trace native operations
import { Systrace } from 'react-native';

Systrace.beginEvent('MyOperation');
doSomethingExpensive();
Systrace.endEvent();

// Hermes Sampling Profiler
// Dev Menu → Start/Stop Sampling Profiler
// → Download profile → Open in Chrome DevTools Performance tab

// React Native Performance (react-native-performance)
import performance from 'react-native-performance';

performance.mark('screenStart');
// ... render screen
performance.mark('screenReady');
performance.measure('Screen Load', 'screenStart', 'screenReady');
```

**Error tracking trong production:**

```typescript
// Sentry — lỗi phổ biến nhất
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'https://your-dsn@sentry.io/project',
  tracesSampleRate: 0.2,
  environment: __DEV__ ? 'development' : 'production',
});

// Wrap App
export default Sentry.wrap(App);

// Manual error capture
try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { screen: 'ProductDetail' },
    extra: { productId },
  });
}
```

---

## 15. Handling Keyboard

**[Mid]**

> **EN:** How do you handle keyboard in React Native? Explain KeyboardAvoidingView, keyboard events, and preventing keyboard overlap with content.
>
> **VI:** Xử lý keyboard trong React Native như thế nào? Giải thích KeyboardAvoidingView, keyboard events và tránh keyboard che content?

**Trả lời:**

Keyboard là một trong những vấn đề phức tạp nhất của React Native, đặc biệt khi behavior khác nhau giữa iOS và Android.

**KeyboardAvoidingView — giải pháp cơ bản:**

```typescript
import { KeyboardAvoidingView, Platform, ScrollView, StyleSheet } from 'react-native';

function LoginScreen() {
  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      // iOS: 'padding' hoặc 'height' (padding thường tốt hơn)
      // Android: thường không cần vì windowSoftInputMode handle nó
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      // Offset nếu có header
      keyboardVerticalOffset={Platform.OS === 'ios' ? 64 : 0}
    >
      <ScrollView
        contentContainerStyle={{ flexGrow: 1 }}
        keyboardShouldPersistTaps="handled" // tap ngoài keyboard không dismiss
      >
        <View style={styles.form}>
          <TextInput placeholder="Email" />
          <TextInput placeholder="Password" secureTextEntry />
          <Button title="Login" onPress={handleLogin} />
        </View>
      </ScrollView>
    </KeyboardAvoidingView>
  );
}
```

**Keyboard events:**

```typescript
import { Keyboard, KeyboardEvent } from 'react-native';

function ChatScreen() {
  const [keyboardHeight, setKeyboardHeight] = useState(0);

  useEffect(() => {
    const showSubscription = Keyboard.addListener(
      // iOS: keyboardWillShow (trước animation)
      // Android: keyboardDidShow (sau animation)
      'keyboardDidShow',
      (event: KeyboardEvent) => {
        setKeyboardHeight(event.endCoordinates.height);
      }
    );

    const hideSubscription = Keyboard.addListener('keyboardDidHide', () => {
      setKeyboardHeight(0);
    });

    return () => {
      showSubscription.remove();
      hideSubscription.remove();
    };
  }, []);

  // Dismiss keyboard khi tap ngoài
  const dismissKeyboard = () => Keyboard.dismiss();

  return (
    <Pressable style={{ flex: 1 }} onPress={dismissKeyboard}>
      <MessageList />
      {/* Input luôn ở trên keyboard */}
      <View style={{ paddingBottom: keyboardHeight }}>
        <MessageInput />
      </View>
    </Pressable>
  );
}
```

**react-native-keyboard-controller (recommended):**

```typescript
// Thư viện tốt nhất cho keyboard handling phức tạp
// Dùng Reanimated — smooth và chính xác theo animation của keyboard
import { KeyboardAwareScrollView } from 'react-native-keyboard-controller';

function ComplexForm() {
  return (
    // Tự động scroll để focused TextInput không bị keyboard che
    <KeyboardAwareScrollView
      bottomOffset={20} // khoảng cách từ input đến top of keyboard
    >
      <TextInput placeholder="Name" />
      <TextInput placeholder="Email" />
      <TextInput placeholder="Bio" multiline />
      <TextInput placeholder="Phone" keyboardType="phone-pad" />
    </KeyboardAwareScrollView>
  );
}
```

**useAnimatedKeyboard — animate theo keyboard với Reanimated:**

```typescript
import { useAnimatedKeyboard, useAnimatedStyle } from 'react-native-reanimated';
import Animated from 'react-native-reanimated';

function ChatInput() {
  const keyboard = useAnimatedKeyboard();

  // Style tự động follow keyboard animation (60fps, không jank)
  const translateStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: -keyboard.height.value }],
  }));

  return (
    <Animated.View style={[styles.inputBar, translateStyle]}>
      <TextInput placeholder="Type a message..." />
      <SendButton />
    </Animated.View>
  );
}
```

**Android windowSoftInputMode:**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<activity
  android:name=".MainActivity"
  android:windowSoftInputMode="adjustResize">
  <!-- adjustResize: resize layout khi keyboard hiện (thường dùng) -->
  <!-- adjustPan: pan toàn bộ layout lên (không resize) -->
  <!-- adjustNothing: không làm gì (phải tự xử lý) -->
</activity>
```

**TextInput focus management:**

```typescript
function MultiFieldForm() {
  const emailRef = useRef<TextInput>(null);
  const passwordRef = useRef<TextInput>(null);
  const confirmRef = useRef<TextInput>(null);

  return (
    <View>
      <TextInput
        ref={emailRef}
        placeholder="Email"
        returnKeyType="next"
        // Khi nhấn "Next" trên keyboard → focus field tiếp theo
        onSubmitEditing={() => passwordRef.current?.focus()}
        blurOnSubmit={false} // Không dismiss keyboard khi submit
      />
      <TextInput
        ref={passwordRef}
        placeholder="Password"
        secureTextEntry
        returnKeyType="next"
        onSubmitEditing={() => confirmRef.current?.focus()}
        blurOnSubmit={false}
      />
      <TextInput
        ref={confirmRef}
        placeholder="Confirm Password"
        secureTextEntry
        returnKeyType="done"
        onSubmitEditing={handleSubmit} // Submit form khi nhấn "Done"
      />
    </View>
  );
}
```

---

*Tài liệu này được tối ưu cho phỏng vấn React Native Engineer với 5+ năm kinh nghiệm. Cập nhật lần cuối: 2025.*
