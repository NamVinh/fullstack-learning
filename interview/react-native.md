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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "You've been a React web developer. Why would React Native feel familiar, and where would you get tripped up first?"**

**Model Answer:**
"The familiar parts are everything React: components, hooks, JSX, state management, context, lifecycle — all of that carries over directly. I can use the same patterns for fetching data, the same libraries like React Query or Zustand, and write components the same way.

Where I'd get tripped up: first, there's no DOM. No `div`, no `span`, no CSS classes. Every piece of text must live inside a `<Text>` component — React Native throws an error if you put a string directly inside a `<View>`. That trips up web developers constantly in the first week.

Second, styling is completely different. You use JavaScript objects through `StyleSheet.create`, not CSS. There's no cascade, no inheritance between components, only a limited subset of CSS properties, and the unit is density-independent pixels, not `px`. Flexbox works but `flexDirection` defaults to `column` instead of `row`, which throws off your mental model.

Third, navigation is much more manual. There's no browser URL bar, no back-forward history — you set up a navigator stack explicitly with something like React Navigation. And platform-specific quirks show up constantly: shadow styles work differently on iOS vs Android, keyboard behavior is different, the safe area insets around the notch need to be handled manually.

So I'd say the learning curve is not React — it's unlearning the browser APIs and learning the native platform APIs instead."

**Trả lời (Tiếng Việt):**
"Những thứ quen thuộc là tất cả những gì thuộc về React: components, hooks, JSX, state management, context, lifecycle — tất cả đều dùng được y chang. Tôi vẫn dùng cùng một pattern để fetch data, cùng thư viện như React Query hay Zustand, viết component theo cách tương tự.

Điểm dễ bị vấp: đầu tiên là không có DOM. Không có `div`, không có `span`, không có CSS class. Mọi đoạn text đều phải nằm trong component `<Text>` — React Native sẽ báo lỗi nếu bạn đặt chuỗi ký tự trực tiếp trong `<View>`. Cái này làm tripped up developer web liên tục trong tuần đầu.

Thứ hai, styling hoàn toàn khác. Phải dùng JavaScript object thông qua `StyleSheet.create`, không phải CSS. Không có cascade, không có inheritance giữa các component, chỉ hỗ trợ một subset CSS properties, và đơn vị là density-independent pixels chứ không phải `px`. Flexbox vẫn dùng được nhưng `flexDirection` mặc định là `column` thay vì `row`, cái đó đảo lộn mental model của mình.

Thứ ba, navigation phải setup thủ công nhiều hơn. Không có URL bar của browser, không có lịch sử back-forward — bạn phải tự setup navigator stack bằng React Navigation. Và các quirk theo nền tảng xuất hiện liên tục: shadow style khác nhau giữa iOS và Android, keyboard behavior khác nhau, safe area insets quanh notch phải xử lý thủ công.

Tóm lại, đường cong học không phải ở React mà là ở việc 'unlearn' browser APIs và học native platform APIs thay thế."

---

**Q2 (Mid): "How does React Native actually render components to the screen? Walk me through the architecture."**

**Model Answer:**
"In the old architecture, React Native ran three threads: the JavaScript thread where your React code ran, the native main thread where UI rendering happened, and a shadow thread for layout calculations using the Yoga layout engine.

Your React code produces a virtual DOM of components. That gets serialized as JSON messages and sent over the 'bridge' to the shadow thread, which computed layouts, then passed instructions to the main thread to create or update actual native views — `UIView` on iOS, `View` on Android. Every interaction and update required crossing this bridge asynchronously.

The problem was the bridge was the bottleneck. Animations would jank if the JS thread was busy because they had to wait for bridge messages. Synchronous calls to native code were impossible. And serializing large data structures to JSON just to move them between threads was wasteful.

The New Architecture — which is the default from React Native 0.74 — replaces the bridge with JSI, JavaScript Interface. JSI lets JavaScript hold direct C++ references to native objects. No serialization, no async round trips for things that should be synchronous. TurboModules are native modules that load lazily and can be called synchronously. Fabric is the new renderer that does layout in C++ and can synchronously read layout measurements.

The result is better animation performance, React 18 concurrent mode support, and faster startup because native modules don't all load upfront."

**Trả lời (Tiếng Việt):**
"Trong old architecture, React Native chạy 3 thread: JavaScript thread nơi code React của bạn chạy, native main thread nơi UI rendering xảy ra, và shadow thread để tính toán layout bằng Yoga layout engine.

Code React của bạn tạo ra virtual DOM của các component. Cái đó được serialize thành JSON messages và gửi qua 'bridge' đến shadow thread, thread này tính toán layout rồi truyền instructions đến main thread để tạo hoặc cập nhật các native view thực sự — `UIView` trên iOS, `View` trên Android. Mỗi tương tác và cập nhật đều phải vượt qua bridge này theo kiểu bất đồng bộ.

Vấn đề là bridge chính là điểm bottleneck. Animation bị jank nếu JS thread đang bận vì phải chờ bridge messages. Gọi synchronous đến native code là không thể. Serialize data structure lớn sang JSON chỉ để di chuyển chúng giữa các thread là rất lãng phí.

New Architecture — mặc định từ React Native 0.74 — thay thế bridge bằng JSI, tức là JavaScript Interface. JSI cho phép JavaScript giữ tham chiếu C++ trực tiếp đến native objects. Không serialize, không async round trip cho những thứ cần synchronous. TurboModules là native modules load lazily và có thể gọi synchronously. Fabric là renderer mới thực hiện layout trong C++ và có thể đọc layout measurements một cách synchronous.

Kết quả là animation performance tốt hơn, hỗ trợ React 18 concurrent mode, và startup nhanh hơn vì native modules không cần load hết lúc khởi động."

---

**Q3 (Mid): "What's the difference between React Navigation and a native navigator? When would you choose one over the other?"**

**Model Answer:**
"React Navigation is JavaScript-based navigation. The navigation stack and animations are implemented in React Native itself, using Reanimated and Gesture Handler under the hood. It's cross-platform with consistent APIs and very customizable.

A native navigator, like React Native Navigation from Wix, uses the actual native navigation components — `UINavigationController` on iOS, `Fragment` and `Activity` on Android. The animations are literally the system animations that iOS and Android users are used to. It can feel more native to platform-savvy users.

The trade-off: React Navigation is much easier to set up, has excellent TypeScript support, a huge community, and its Expo integration is seamless. The animations with Reanimated are smooth and customizable. React Native Navigation is harder to set up — you need to link native modules manually — and requires more platform-specific thinking.

In practice, I'd choose React Navigation for almost every project. The gap in 'native feel' has narrowed significantly as Reanimated has improved, and the developer experience and community support are far better. React Native Navigation makes more sense if you're integrating into an existing native app where you need to match the exact system navigation behavior, or if your app is extremely navigation-heavy and the 'close to real native' feel is a hard product requirement."

**Trả lời (Tiếng Việt):**
"React Navigation là navigation dựa trên JavaScript. Navigation stack và animation được implement trong React Native, sử dụng Reanimated và Gesture Handler bên dưới. Nó cross-platform với API nhất quán và rất dễ customize.

Native navigator, như React Native Navigation của Wix, sử dụng đúng các native navigation components thực sự — `UINavigationController` trên iOS, `Fragment` và `Activity` trên Android. Animation là đúng system animation mà người dùng iOS và Android quen dùng. Cảm giác native hơn với những người dùng tinh tế.

Trade-off: React Navigation dễ setup hơn nhiều, TypeScript support xuất sắc, cộng đồng lớn, và tích hợp với Expo rất liền mạch. Animation với Reanimated mượt và dễ customize. React Native Navigation khó setup hơn — cần link native modules thủ công — và đòi hỏi phải suy nghĩ theo kiểu platform-specific nhiều hơn.

Trong thực tế, tôi sẽ chọn React Navigation cho gần như mọi dự án. Khoảng cách về 'cảm giác native' đã thu hẹp đáng kể khi Reanimated cải thiện, và developer experience cũng như community support tốt hơn nhiều. React Native Navigation có ý nghĩa hơn khi bạn tích hợp vào một app native đã có sẵn và cần khớp đúng với system navigation behavior, hoặc khi app của bạn cực kỳ nặng về navigation và 'gần với native thực sự' là yêu cầu sản phẩm bắt buộc."

---

**Q4 (Senior): "A user on your team says you should rewrite a React web app as a React Native app to 'get both platforms for free.' What's your response?"**

**Model Answer:**
"The shared knowledge is real — React skills transfer, state management patterns work the same, and the business logic can often be shared. But 'for free' overstates it significantly.

The UI layer is essentially a full rewrite. Every HTML element becomes a native component, all CSS has to be rewritten as StyleSheet objects, web-specific patterns like hover states, focus rings, and relative URLs don't exist. If the web app has any significant CSS or styling, that's not shared.

APIs diverge significantly. `localStorage` becomes AsyncStorage or MMKV. `window` and `document` don't exist. Routing is completely different. Web-specific libraries — many charting, animation, or rich text libraries — don't work in React Native. You'll spend a lot of time finding native equivalents.

Platform-specific quirks multiply. Instead of dealing with browser differences, you now deal with iOS vs Android differences, plus different iOS and Android versions, different screen sizes and pixel densities, notches, safe areas, and hardware back buttons.

There's also the build and deployment overhead — dealing with Xcode, Android Studio, code signing, App Store review cycles.

A better framing is: React Native significantly lowers the barrier to mobile development for a React team compared to learning Swift and Kotlin from scratch. But it's still a meaningful investment — probably a 60% rewrite, not a 10% one. If the business case for a native mobile app is strong, it's the right choice. But 'we have a web app, let's get mobile for free' usually leads to underestimating the effort."

**Trả lời (Tiếng Việt):**
"Kiến thức chung là có thật — kỹ năng React có thể dùng được, pattern state management hoạt động tương tự, và business logic thường có thể dùng lại. Nhưng 'miễn phí' là nói quá đáng kể.

Tầng UI về cơ bản là viết lại hoàn toàn. Mọi HTML element trở thành native component, tất cả CSS phải viết lại thành StyleSheet object, các pattern dành riêng cho web như hover states, focus rings, và relative URL không tồn tại. Nếu web app có CSS hoặc styling đáng kể, phần đó không được dùng lại.

API phân kỳ đáng kể. `localStorage` trở thành AsyncStorage hoặc MMKV. `window` và `document` không tồn tại. Routing hoàn toàn khác. Các thư viện dành riêng cho web — nhiều thư viện charting, animation, hay rich text — không hoạt động trong React Native. Bạn sẽ tốn nhiều thời gian tìm native equivalent.

Quirk theo nền tảng nhân lên. Thay vì phải lo browser differences, bạn giờ phải xử lý iOS vs Android differences, cộng thêm các phiên bản iOS và Android khác nhau, kích thước màn hình và pixel density khác nhau, notch, safe area, và hardware back button.

Còn có overhead về build và deployment — phải xử lý Xcode, Android Studio, code signing, App Store review cycles.

Cách nói chính xác hơn là: React Native giảm đáng kể rào cản vào mobile development cho một team React so với việc học Swift và Kotlin từ đầu. Nhưng đó vẫn là một khoản đầu tư có ý nghĩa — có lẽ là viết lại 60%, không phải 10%. Nếu business case cho native mobile app đủ mạnh, đó là lựa chọn đúng. Nhưng 'mình có web app rồi, lấy mobile miễn phí thôi' thường dẫn đến việc đánh giá thấp công sức."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "We have a list of 5000 products. Using FlatList, users are reporting jank when scrolling. Where would you start debugging?"**

**Model Answer:**
"First I'd open the Performance Monitor in the Dev Menu to see which thread is dropping frames. If the JS FPS is dropping, the issue is computation on the JavaScript thread — re-renders, heavy calculations in render. If UI FPS is dropping but JS FPS is fine, the issue is in the native rendering layer.

For JS thread issues, the first thing I'd check is whether the renderItem function is creating new function references on every parent render. Wrapping renderItem in `useCallback` and the item component in `React.memo` prevents unnecessary re-renders.

Next I'd check if the item component is doing any expensive work during render — complex calculations, large object spreads. Anything that can be memoized or moved outside the component should be.

Then I'd look at whether the list items have fixed height. If they do, adding `getItemLayout` is a big win — it lets FlatList calculate scroll positions without measuring every item. Without it, the list has to render items it hasn't shown yet just to know the scroll offset.

I'd also review the `windowSize` and `maxToRenderPerBatch` props. The defaults are conservative — 21 windows and 10 items per batch. For a product list with images, I'd reduce `windowSize` to around 5 to 10, which means fewer off-screen items are mounted.

If those changes aren't enough, I'd seriously consider switching to Shopify's FlashList. It recycles native views instead of unmounting and remounting components, which is fundamentally more efficient — typically around 10x faster for long lists."

**Trả lời (Tiếng Việt):**
"Đầu tiên tôi sẽ mở Performance Monitor trong Dev Menu để xem thread nào đang bị drop frame. Nếu JS FPS giảm, vấn đề là tính toán trên JavaScript thread — re-render, tính toán nặng trong render. Nếu UI FPS giảm nhưng JS FPS vẫn ổn, vấn đề nằm ở native rendering layer.

Với JS thread issues, điều đầu tiên tôi kiểm tra là liệu hàm renderItem có đang tạo function reference mới mỗi lần parent render không. Wrap renderItem bằng `useCallback` và component item bằng `React.memo` để tránh re-render không cần thiết.

Tiếp theo tôi kiểm tra xem component item có đang làm công việc nặng trong render không — tính toán phức tạp, spread object lớn. Bất cứ thứ gì có thể memoize hoặc đưa ra ngoài component thì nên làm.

Sau đó tôi xem items có chiều cao cố định không. Nếu có, thêm `getItemLayout` là một cải thiện lớn — nó cho FlatList tính vị trí scroll mà không cần render từng item. Không có nó, list phải render những items chưa hiển thị chỉ để biết scroll offset.

Tôi cũng xem lại props `windowSize` và `maxToRenderPerBatch`. Default khá bảo thủ — 21 windows và 10 items mỗi batch. Với danh sách sản phẩm có ảnh, tôi sẽ giảm `windowSize` xuống khoảng 5 đến 10, tức là ít off-screen items được mount hơn.

Nếu các thay đổi đó vẫn không đủ, tôi sẽ nghiêm túc xem xét chuyển sang FlashList của Shopify. Nó recycle native views thay vì unmount và remount components, hiệu quả hơn về bản chất — thường nhanh hơn khoảng 10 lần với danh sách dài."

---

**Q2 (Mid): "What is `getItemLayout` and when should you NOT use it?"**

**Model Answer:**
"`getItemLayout` is a prop that lets you tell FlatList the exact dimensions of each item without rendering them. You provide a function that takes the data array and an index, and returns the item's length — height for vertical lists — its offset from the top of the list, and its index.

This is really valuable when items have a fixed, known height. FlatList can calculate any item's scroll position instantly without rendering intermediate items, which makes `scrollToIndex` and `initialScrollIndex` accurate and fast. Without `getItemLayout`, if you try to scroll to index 500, the list has to render items 0 through 499 to figure out where 500 is.

You should NOT use it if your items have variable heights. If you calculate wrong offsets, scrolling will be incorrect — items will visually jump or overlap when the list tries to scroll to a position that doesn't match the actual rendered position. It's better to have a slightly slower list than broken scroll behavior.

Also don't use it if items can resize dynamically — for example, expandable items or items that change height based on loaded content. Stick to `getItemLayout` only when you can guarantee the height is constant and known at render time."

**Trả lời (Tiếng Việt):**
"`getItemLayout` là một prop cho phép bạn nói với FlatList kích thước chính xác của từng item mà không cần render chúng. Bạn cung cấp một hàm nhận data array và một index, trả về chiều dài của item — height cho vertical list — offset của nó tính từ đầu list, và index của nó.

Cái này rất có giá trị khi items có chiều cao cố định và đã biết. FlatList có thể tính vị trí scroll của bất kỳ item nào ngay lập tức mà không cần render các item trung gian, làm cho `scrollToIndex` và `initialScrollIndex` trở nên chính xác và nhanh. Không có `getItemLayout`, nếu bạn thử scroll đến index 500, list phải render item 0 đến 499 để biết vị trí của 500.

Bạn KHÔNG NÊN dùng nó nếu items có chiều cao thay đổi. Nếu bạn tính offset sai, scroll sẽ bị lỗi — items sẽ nhảy hoặc chồng lên nhau khi list cố gắng scroll đến vị trí không khớp với vị trí render thực tế. Tốt hơn là có list hơi chậm còn hơn là scroll bị vỡ.

Cũng đừng dùng nếu items có thể resize dynamically — ví dụ như expandable items hoặc items thay đổi chiều cao dựa trên content đã load. Chỉ dùng `getItemLayout` khi bạn có thể đảm bảo chiều cao là cố định và biết trước lúc render."

---

**Q3 (Senior): "Explain the difference between `windowSize`, `maxToRenderPerBatch`, and `initialNumToRender`. How do you tune these for a product catalog?"**

**Model Answer:**
"These three props control different aspects of how FlatList manages its render work.

`initialNumToRender` is how many items to render on the first paint. It should be enough to fill the screen — so if each item is 100 points tall and the screen is 800 points, you'd want at least 8, but I'd set it to 12 or 15 to have a small buffer above the fold. Setting it too high means slower initial render; too low means you see a flash of empty space right after mount.

`maxToRenderPerBatch` is how many items FlatList renders in each incremental batch as the user scrolls. The default is 10. Lower values mean each individual scroll step does less work on the JS thread, so you get smoother frame rates, but more blank space if the user scrolls quickly. Higher values mean content appears faster but each batch takes more JS thread time.

`windowSize` controls how large the render window is, measured in 'screens.' The default of 21 means 10 screens above the visible area and 10 screens below are rendered. For a dense product catalog on a phone with 6GB RAM, I'd bring this down to 5 — 2 screens above, viewport, 2 screens below. This dramatically reduces the number of mounted components and memory usage, at the cost of more blank space if the user flings the scroll very quickly.

For a product catalog specifically, I'd also pair this with image lazy loading — use `FastImage` or `expo-image` which have better caching, and only start loading images when items enter the window. That way reducing the window size doesn't cause visible image popping either."

**Trả lời (Tiếng Việt):**
"Ba props này kiểm soát các khía cạnh khác nhau của cách FlatList quản lý công việc render.

`initialNumToRender` là số items render trong lần paint đầu tiên. Nên đủ để fill màn hình — nếu mỗi item cao 100 points và màn hình cao 800 points, bạn cần ít nhất 8, nhưng tôi thường set là 12 hoặc 15 để có một buffer nhỏ trên fold. Set quá cao sẽ làm render ban đầu chậm hơn; set quá thấp sẽ thấy flash khoảng trắng ngay sau khi mount.

`maxToRenderPerBatch` là số items FlatList render trong mỗi batch tăng dần khi người dùng scroll. Default là 10. Giá trị thấp hơn có nghĩa là mỗi bước scroll riêng lẻ làm ít việc hơn trên JS thread, nên frame rate mượt hơn, nhưng nhiều khoảng trắng hơn nếu người dùng scroll nhanh. Giá trị cao hơn có nghĩa là content xuất hiện nhanh hơn nhưng mỗi batch tốn nhiều JS thread time hơn.

`windowSize` kiểm soát cửa sổ render lớn đến đâu, tính bằng 'số màn hình'. Default là 21, tức là 10 màn hình phía trên vùng nhìn thấy và 10 màn hình phía dưới đều được render. Với catalog sản phẩm dày đặc trên điện thoại 6GB RAM, tôi sẽ giảm xuống còn 5 — 2 màn hình phía trên, viewport, 2 màn hình phía dưới. Điều này giảm mạnh số components đang được mount và mức sử dụng bộ nhớ, đánh đổi bằng nhiều khoảng trắng hơn nếu người dùng fling scroll rất nhanh.

Với catalog sản phẩm cụ thể, tôi cũng kết hợp với image lazy loading — dùng `FastImage` hoặc `expo-image` với caching tốt hơn, và chỉ bắt đầu load ảnh khi items vào cửa sổ. Cách đó giảm window size mà không gây ra hiện tượng ảnh bị nhấp nháy."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "How does navigation work in React Native? How is it different from React Router on the web?"**

**Model Answer:**
"On the web, navigation is URL-based. React Router maps URL paths to components, and the browser's built-in history API handles back-forward. You can deep link directly to any page, share URLs, and bookmarks just work.

In React Native, there's no URL bar. Navigation is stack-based — you push screens onto a stack and pop them off. React Navigation is the standard library. You define navigators and screens in JavaScript, and React Navigation manages the history stack and transitions in React Native.

The most common setup is a native stack navigator for the main screen flow, and a bottom tab navigator for the app's top-level sections. You nest them — tabs are a screen in the stack, each tab has its own stack, and so on.

One difference I appreciate: React Navigation with TypeScript gives you strongly-typed params. You define a type that maps screen names to their required params, and TypeScript will error if you try to navigate to a screen without providing the right params. On the web with React Router, passing params through route state isn't typed the same way.

Deep linking is supported but requires explicit configuration — you map URL patterns to screen names and React Navigation handles parsing the params out of the URL."

**Trả lời (Tiếng Việt):**
"Trên web, navigation dựa trên URL. React Router map đường dẫn URL đến components, và browser's history API xử lý back-forward. Bạn có thể deep link trực tiếp đến bất kỳ trang nào, chia sẻ URL, và bookmark hoạt động tự nhiên.

Trong React Native, không có URL bar. Navigation dựa trên stack — bạn push màn hình lên stack và pop chúng xuống. React Navigation là thư viện chuẩn. Bạn định nghĩa navigators và screens bằng JavaScript, và React Navigation quản lý history stack và transition trong React Native.

Setup phổ biến nhất là native stack navigator cho luồng màn hình chính, và bottom tab navigator cho các section cấp cao nhất của app. Bạn lồng chúng vào nhau — tabs là một màn hình trong stack, mỗi tab có stack riêng của nó, và cứ thế.

Một điểm tôi thích: React Navigation với TypeScript cho bạn typed params rõ ràng. Bạn định nghĩa một type map tên màn hình đến params bắt buộc của chúng, và TypeScript sẽ báo lỗi nếu bạn navigate đến một màn hình mà không cung cấp đúng params. Trên web với React Router, truyền params qua route state không được typed theo cách tương tự.

Deep linking được hỗ trợ nhưng cần cấu hình rõ ràng — bạn map các pattern URL đến tên màn hình và React Navigation xử lý việc parse params ra từ URL."

---

**Q2 (Mid): "How do you pass data between screens in React Navigation? What are the pitfalls of passing complex objects as params?"**

**Model Answer:**
"You pass params in the second argument to `navigation.navigate`: `navigation.navigate('ProductDetail', { productId: '123', productName: 'iPhone 15' })`. In the destination screen you access them through `route.params`.

The pitfall with complex objects is that React Navigation serializes params to support deep linking and state persistence. If you pass a full product object with images, nested categories, and methods, you're creating serialization overhead on every navigation, and if those params are stored in navigation state for deep linking, you've made your navigation state very heavy.

The recommended pattern is to pass only identifiers — pass `productId`, not the full product object. The destination screen fetches or looks up the full data from your data store using that ID. This is cleaner and more resilient: if the screen is opened from a deep link, it has an ID to fetch from, not a serialized object that might be stale.

Another pitfall: non-serializable values like functions or class instances as params. React Navigation will warn about these and they won't survive deep linking or state restoration. Callbacks between screens should go through navigation events, a shared context, or a state management store, not as params.

For data that needs to be shared across many screens, I put it in a Zustand store or React Query cache rather than threading it through navigation params."

**Trả lời (Tiếng Việt):**
"Bạn truyền params ở argument thứ hai của `navigation.navigate`: `navigation.navigate('ProductDetail', { productId: '123', productName: 'iPhone 15' })`. Trong màn hình đích bạn truy cập chúng qua `route.params`.

Pitfall với object phức tạp là React Navigation serialize params để hỗ trợ deep linking và state persistence. Nếu bạn truyền một object product đầy đủ với ảnh, nested categories, và methods, bạn đang tạo serialization overhead mỗi lần navigate, và nếu params đó được lưu trong navigation state cho deep linking, bạn đã làm navigation state rất nặng.

Pattern được khuyến nghị là chỉ truyền identifiers — truyền `productId`, không phải object product đầy đủ. Màn hình đích fetch hoặc lookup data đầy đủ từ data store dùng ID đó. Cách này clean hơn và resilient hơn: nếu màn hình được mở từ deep link, nó có ID để fetch, không phải object serialize có thể đã stale.

Một pitfall khác: các giá trị non-serializable như functions hay class instances làm params. React Navigation sẽ warn về những cái này và chúng sẽ không tồn tại qua deep linking hay state restoration. Callbacks giữa các màn hình nên đi qua navigation events, shared context, hoặc state management store, không phải params.

Với data cần chia sẻ qua nhiều màn hình, tôi đặt nó trong Zustand store hoặc React Query cache thay vì threading qua navigation params."

---

**Q3 (Mid): "How do you implement deep linking in React Navigation? Give me a concrete example."**

**Model Answer:**
"Deep linking lets external URLs open specific screens in your app. You configure it in the `linking` prop on `NavigationContainer`.

The config has two parts: `prefixes` — the URL schemes your app handles, like `myapp://` for the custom scheme and `https://myapp.com` for universal links — and `config`, which maps path patterns to screen names.

For example, if I want `myapp://product/123` to open the `ProductDetail` screen with `productId` set to `123`, I'd write:

```
config: {
  screens: {
    Home: '',
    ProductDetail: 'product/:productId',
    Cart: 'cart',
  }
}
```

React Navigation automatically parses the `:productId` segment from the URL and passes it as a route param. So the screen gets `route.params.productId` equal to `'123'`.

For iOS you register the URL scheme in `Info.plist` and configure universal links through an Apple App Site Association file on your web server. Android uses intent filters in `AndroidManifest.xml`. With Expo, you configure this in `app.json` and the managed workflow handles most of the boilerplate.

One thing to test carefully: what happens when the deep link arrives and the app is in a killed state. React Navigation handles this correctly — it reconstructs the navigation state from the URL — but you need to make sure your screens don't crash if opened without the normal preceding screens in the stack."

**Trả lời (Tiếng Việt):**
"Deep linking cho phép URL bên ngoài mở các màn hình cụ thể trong app của bạn. Bạn cấu hình nó trong prop `linking` của `NavigationContainer`.

Config có hai phần: `prefixes` — các URL scheme mà app của bạn xử lý, như `myapp://` cho custom scheme và `https://myapp.com` cho universal links — và `config`, map các pattern đường dẫn đến tên màn hình.

Ví dụ, nếu tôi muốn `myapp://product/123` mở màn hình `ProductDetail` với `productId` được set là `123`, tôi sẽ viết:

```
config: {
  screens: {
    Home: '',
    ProductDetail: 'product/:productId',
    Cart: 'cart',
  }
}
```

React Navigation tự động parse segment `:productId` từ URL và truyền nó như route param. Vậy màn hình nhận được `route.params.productId` bằng `'123'`.

Với iOS bạn đăng ký URL scheme trong `Info.plist` và cấu hình universal links thông qua Apple App Site Association file trên web server của bạn. Android dùng intent filters trong `AndroidManifest.xml`. Với Expo, bạn cấu hình trong `app.json` và managed workflow xử lý hầu hết boilerplate.

Một thứ cần test kỹ: điều gì xảy ra khi deep link đến và app đang ở trạng thái bị kill. React Navigation xử lý đúng — nó reconstruct navigation state từ URL — nhưng bạn cần đảm bảo các màn hình không crash khi được mở mà không có các màn hình trước đó bình thường trong stack."

---

**Q4 (Senior): "How would you navigate to a screen from outside the component tree — for example, from a push notification handler?"**

**Model Answer:**
"Since React Navigation lives inside the component tree, you normally call `useNavigation()` inside a component. But notification handlers, background tasks, and auth callbacks run outside the component tree, so you can't use the hook.

The solution is a navigation ref. React Navigation provides `createNavigationContainerRef()`. You create the ref outside the component tree — in its own module — and pass it to `NavigationContainer` via the `ref` prop.

```typescript
// navigationRef.ts
export const navigationRef = createNavigationContainerRef<RootStackParamList>();

// App.tsx
<NavigationContainer ref={navigationRef}>

// notificationHandler.ts
import { navigationRef } from './navigationRef';

if (navigationRef.isReady()) {
  navigationRef.navigate('ProductDetail', { productId: data.id });
}
```

The `isReady()` check is important — if the app just launched, the navigator might not be mounted yet. In that case you store the pending navigation action and dispatch it once the navigator is ready, typically using a `useEffect` in App.tsx that runs when the navigator mounts.

For push notifications specifically, you also need to handle the case where the app was in a killed state and launched by a notification tap — you check `getInitialNotification()` or the equivalent, and navigate after the navigator is ready. The sequence is: app launches, navigator mounts, `isReady()` becomes true, then you dispatch the pending navigation."

**Trả lời (Tiếng Việt):**
"Vì React Navigation nằm trong component tree, bạn thường gọi `useNavigation()` bên trong component. Nhưng notification handlers, background tasks, và auth callbacks chạy ngoài component tree, nên không thể dùng hook được.

Giải pháp là navigation ref. React Navigation cung cấp `createNavigationContainerRef()`. Bạn tạo ref bên ngoài component tree — trong module riêng của nó — và truyền nó vào `NavigationContainer` qua prop `ref`.

```typescript
// navigationRef.ts
export const navigationRef = createNavigationContainerRef<RootStackParamList>();

// App.tsx
<NavigationContainer ref={navigationRef}>

// notificationHandler.ts
import { navigationRef } from './navigationRef';

if (navigationRef.isReady()) {
  navigationRef.navigate('ProductDetail', { productId: data.id });
}
```

Check `isReady()` là quan trọng — nếu app vừa launch, navigator có thể chưa được mount. Trong trường hợp đó bạn lưu pending navigation action và dispatch nó khi navigator ready, thường dùng `useEffect` trong App.tsx chạy khi navigator mount.

Với push notifications cụ thể, bạn cũng cần xử lý trường hợp app ở trạng thái bị kill và được launch bởi notification tap — bạn check `getInitialNotification()` hay tương đương, và navigate sau khi navigator đã sẵn sàng. Trình tự là: app launch, navigator mount, `isReady()` trở thành true, sau đó bạn dispatch pending navigation."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Senior): "What problem does the New Architecture solve? Can you explain JSI in plain terms?"**

**Model Answer:**
"The old architecture had a fundamental bottleneck: the JavaScript bridge. Every time JavaScript needed to talk to native code — to update a UI, call a native API, do a layout calculation — it had to serialize the data as JSON, pass it over an asynchronous message queue to the native side, which then deserialized it. The same thing happened for responses going back.

This meant two things: first, you couldn't make synchronous calls to native from JavaScript. Everything was callback-based. Second, for animations that needed to respond to gestures at 60fps, if the JS thread was busy — doing a list re-render, running a heavy calculation — the bridge messages backed up and animations janked.

JSI, JavaScript Interface, replaces the bridge with direct C++ bindings. JavaScript objects in Hermes can hold references to C++ objects. When you call a native method through JSI, you're literally calling a C++ function through a function pointer — no serialization, no async queue, no round trip.

The analogy I use: the old bridge was like communicating between two offices by printing emails, walking them to a mail room, and having someone carry them across the building. JSI is like both offices sharing the same file system — you write to a file and the other side reads it directly, in memory.

This enables TurboModules — native modules that can be called synchronously and load lazily. And it enables Fabric, the new renderer, to do layout calculations in C++ synchronously, which lets React 18's concurrent mode work properly in React Native."

**Trả lời (Tiếng Việt):**
"Old architecture có một điểm bottleneck cơ bản: JavaScript bridge. Mỗi khi JavaScript cần nói chuyện với native code — để update UI, gọi native API, tính toán layout — nó phải serialize data thành JSON, truyền qua một async message queue đến phía native, bên đó deserialize lại. Chuyện tương tự xảy ra cho response đi ngược lại.

Điều này có nghĩa là hai thứ: thứ nhất, bạn không thể gọi synchronous đến native từ JavaScript. Tất cả đều dựa trên callback. Thứ hai, với animation cần phản hồi gesture ở 60fps, nếu JS thread bận — đang re-render list, chạy tính toán nặng — bridge messages bị tắc nghẽn và animation bị jank.

JSI, JavaScript Interface, thay thế bridge bằng C++ bindings trực tiếp. JavaScript objects trong Hermes có thể giữ tham chiếu đến C++ objects. Khi bạn gọi native method qua JSI, bạn đang thực sự gọi một C++ function qua function pointer — không serialize, không async queue, không round trip.

Analogy tôi hay dùng: old bridge giống như giao tiếp giữa hai văn phòng bằng cách in email, đi bộ đến phòng thư, và nhờ ai đó mang qua tòa nhà. JSI giống như cả hai văn phòng chia sẻ cùng một file system — bạn ghi vào file và bên kia đọc trực tiếp, trong bộ nhớ.

Điều này cho phép TurboModules — native modules có thể gọi synchronously và load lazily. Và cho phép Fabric, renderer mới, thực hiện layout calculations trong C++ synchronously, giúp React 18's concurrent mode hoạt động đúng trong React Native."

---

**Q2 (Senior): "What are TurboModules and how do they differ from the old NativeModules pattern?"**

**Model Answer:**
"In the old architecture, all native modules were registered at startup and available through the `NativeModules` object. Even if your app never used the Camera module or the Bluetooth module, they were all loaded into memory when the app started. This hurt startup time proportionally to how many native modules you had.

TurboModules change this to lazy loading. A TurboModule is only loaded from native code the first time JavaScript actually calls it. If you never use Bluetooth in a given session, that module's native code is never initialized.

The other big difference is typing. Old-style native modules had no type information — you'd call `NativeModules.Camera.takePicture()` and TypeScript had no idea what that function expected or returned. Errors only surfaced at runtime.

TurboModules require a TypeScript spec file that defines the exact interface. A code generator uses that spec to generate native boilerplate for both iOS and Android, and TypeScript enforces the types at call sites. So type errors are caught at compile time.

And because TurboModules work through JSI, they can expose synchronous methods when that's appropriate — something that was impossible with the old bridge where every call was inherently async.

In practice, if you're using Expo's Modules API, a lot of this is abstracted — you write the native code and a TypeScript spec, and the framework handles the JSI plumbing. But understanding what's happening underneath helps when you're debugging or writing a module from scratch."

**Trả lời (Tiếng Việt):**
"Trong old architecture, tất cả native modules được đăng ký lúc startup và có thể truy cập qua object `NativeModules`. Dù app của bạn có bao giờ dùng Camera module hay Bluetooth module không, chúng đều được load vào bộ nhớ khi app khởi động. Điều này làm tăng startup time tỷ lệ thuận với số lượng native modules bạn có.

TurboModules chuyển sang lazy loading. TurboModule chỉ được load từ native code vào lần đầu tiên JavaScript thực sự gọi nó. Nếu bạn không bao giờ dùng Bluetooth trong một session, native code của module đó không bao giờ được khởi tạo.

Điểm khác biệt lớn khác là typing. Old-style native modules không có thông tin type — bạn gọi `NativeModules.Camera.takePicture()` và TypeScript không biết function đó nhận gì hay trả về gì. Lỗi chỉ xuất hiện lúc runtime.

TurboModules yêu cầu một TypeScript spec file định nghĩa interface chính xác. Code generator dùng spec đó để generate native boilerplate cho cả iOS và Android, và TypeScript enforce types tại call sites. Vì vậy lỗi type được phát hiện lúc compile time.

Và vì TurboModules hoạt động qua JSI, chúng có thể expose synchronous methods khi phù hợp — điều không thể với old bridge nơi mọi call đều inherently async.

Trong thực tế, nếu bạn dùng Expo Modules API, nhiều thứ này được trừu tượng hóa rồi — bạn viết native code và TypeScript spec, framework lo phần JSI plumbing. Nhưng hiểu những gì đang xảy ra bên dưới giúp ích khi bạn debug hoặc viết module từ đầu."

---

**Q3 (Senior): "What is Fabric and how does it affect how we write React Native components?"**

**Model Answer:**
"Fabric is the new rendering system in React Native's New Architecture. It replaces the old renderer that coordinated between the JavaScript thread, shadow thread, and main thread via bridge messages.

In the old architecture, when React committed changes to the virtual DOM, those had to travel asynchronously over the bridge to a shadow thread for layout calculations, then asynchronously again to the main thread for actual view creation. This meant the native UI was always slightly behind what JavaScript knew.

With Fabric, the renderer is written in C++ and shared across platforms. The shadow tree — the layout tree — lives in C++ and can be manipulated synchronously from both JavaScript and native. When React commits, it can immediately talk to the C++ renderer, and layout calculations happen synchronously.

The developer-visible impacts are: first, React 18 concurrent features work properly. Transitions, `startTransition`, `useDeferredValue` — all of these require the renderer to support interruption and prioritization, which Fabric enables. In the old architecture, concurrent mode was essentially non-functional.

Second, host component measurements can now be synchronous. If a component needs to know its own dimensions to render, it can get that synchronously rather than waiting for a bridge round trip.

Third, animations tied to native gestures can be truly synchronous — no frame lag between a touch event and the UI response. This is what powers the silky animations in Reanimated 3."

**Trả lời (Tiếng Việt):**
"Fabric là rendering system mới trong React Native's New Architecture. Nó thay thế old renderer phối hợp giữa JavaScript thread, shadow thread, và main thread qua bridge messages.

Trong old architecture, khi React commit changes vào virtual DOM, chúng phải đi bất đồng bộ qua bridge đến shadow thread để tính toán layout, rồi bất đồng bộ một lần nữa đến main thread để tạo view thực sự. Điều này có nghĩa là native UI luôn hơi trễ hơn so với những gì JavaScript biết.

Với Fabric, renderer được viết bằng C++ và chia sẻ across platforms. Shadow tree — cây layout — nằm trong C++ và có thể được thao tác synchronously từ cả JavaScript và native. Khi React commit, nó có thể nói chuyện ngay với C++ renderer, và layout calculations xảy ra synchronously.

Các tác động mà developer nhìn thấy: thứ nhất, React 18 concurrent features hoạt động đúng. Transitions, `startTransition`, `useDeferredValue` — tất cả những thứ này yêu cầu renderer hỗ trợ interruption và prioritization, điều mà Fabric cho phép. Trong old architecture, concurrent mode về cơ bản không hoạt động.

Thứ hai, host component measurements giờ có thể synchronous. Nếu component cần biết kích thước của chính nó để render, nó có thể lấy thông tin đó synchronously thay vì chờ bridge round trip.

Thứ ba, animation gắn với native gestures có thể thực sự synchronous — không có frame lag giữa touch event và UI response. Đây là thứ tạo ra animation mượt mà trong Reanimated 3."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "Our app feels sluggish after navigation. Users are on older Android phones. Where do you start?"**

**Model Answer:**
"First, I'd open the Performance Monitor from the Dev Menu to see which thread is struggling. If the UI FPS drops but JS FPS is fine, the issue is in native rendering — complex view hierarchies, too many layers, overdraw. If JS FPS drops, the issue is JavaScript work — heavy renders, synchronous computations.

For navigation-related sluggishness specifically, a common culprit is rendering heavy content while the navigation animation is still in progress. The solution is `InteractionManager.runAfterInteractions`. You wrap your heavy render in a callback passed to it, and it defers that work until all animations have completed. The screen navigates in smoothly, then the heavy content loads.

On older Android, shadow and elevation styles are expensive. If your UI has many layered views with `elevation`, try reducing the number of elevated surfaces or flattening the view hierarchy.

I'd also check if list components are using proper optimization — `getItemLayout` for fixed-height lists, `React.memo` on item components, `useCallback` on handlers passed as props. On older hardware, unnecessary re-renders are much more visible.

If the animation itself is janky rather than the content, I'd check if the navigation is using `react-native-screens` which is a requirement for the native stack navigator to use native platform animations rather than JavaScript-driven ones. Expo includes it by default, but bare workflow apps sometimes miss this."

**Trả lời (Tiếng Việt):**
"Đầu tiên, tôi sẽ mở Performance Monitor từ Dev Menu để xem thread nào đang gặp vấn đề. Nếu UI FPS giảm nhưng JS FPS ổn, vấn đề là ở native rendering — view hierarchy phức tạp, quá nhiều layer, overdraw. Nếu JS FPS giảm, vấn đề là JavaScript — render nặng, tính toán synchronous.

Với navigation sluggishness cụ thể, thủ phạm phổ biến là render heavy content khi navigation animation vẫn đang chạy. Giải pháp là `InteractionManager.runAfterInteractions`. Bạn wrap heavy render trong callback truyền vào nó, và nó defer công việc đó cho đến khi tất cả animation hoàn thành. Màn hình navigate vào mượt mà, rồi heavy content mới load.

Trên Android cũ, shadow và elevation styles rất đắt. Nếu UI của bạn có nhiều layered views với `elevation`, thử giảm số lượng elevated surfaces hoặc flatten view hierarchy.

Tôi cũng kiểm tra xem list components có dùng optimization đúng không — `getItemLayout` cho fixed-height lists, `React.memo` trên item components, `useCallback` trên handlers truyền làm props. Trên phần cứng cũ, unnecessary re-renders thấy rõ hơn nhiều.

Nếu bản thân animation bị janky thay vì content, tôi kiểm tra xem navigation có dùng `react-native-screens` không — đây là yêu cầu để native stack navigator dùng native platform animations thay vì JavaScript-driven. Expo include mặc định, nhưng bare workflow apps đôi khi bị thiếu."

---

**Q2 (Senior): "Explain the difference between Reanimated and the built-in Animated API. When does each one make sense?"**

**Model Answer:**
"The built-in Animated API drives animations on the JavaScript thread. When you call `Animated.timing`, React Native schedules frame updates on the JS thread and sends style updates to the native layer each frame. If the JS thread gets busy — list rendering, state updates, heavy computations — animation frames get delayed and you see jank.

There's a partial escape: `useNativeDriver: true`. This offloads certain animations to the native thread, but only supports a limited set of properties — `opacity`, `transform`, and a few others. You can't use native driver for `width`, `height`, `backgroundColor`, or `borderRadius`.

Reanimated 3 runs animations as 'worklets' — small functions that execute directly on the UI thread, written in JavaScript but compiled to run natively. The animation logic itself never touches the JS thread. This means animations stay at 60fps even if the JS thread is completely saturated.

Reanimated is the clear choice for gesture-driven animations — when a pan gesture should move a view, or a swipe should trigger a transition. These need to respond in the same frame as the touch event, which requires UI thread execution.

Use the built-in Animated API for simple, non-interactive animations — a loading spinner, a fade-in on mount, a progress bar that doesn't respond to touch. These are simpler to set up and the code is cleaner for basic cases.

Use Reanimated when: the animation responds to gestures, you need to animate properties not supported by the native driver, you need complex interpolations synchronized with scroll position, or when you've profiled and found the Animated API dropping frames."

**Trả lời (Tiếng Việt):**
"Built-in Animated API drive animation trên JavaScript thread. Khi bạn gọi `Animated.timing`, React Native lên lịch frame updates trên JS thread và gửi style updates đến native layer mỗi frame. Nếu JS thread bận — render list, state updates, tính toán nặng — animation frames bị delay và bạn thấy jank.

Có một giải pháp thoát một phần: `useNativeDriver: true`. Cái này offload một số animation nhất định sang native thread, nhưng chỉ hỗ trợ một tập properties hạn chế — `opacity`, `transform`, và một vài cái khác. Bạn không thể dùng native driver cho `width`, `height`, `backgroundColor`, hay `borderRadius`.

Reanimated 3 chạy animation như 'worklets' — các hàm nhỏ thực thi trực tiếp trên UI thread, viết bằng JavaScript nhưng được compile để chạy natively. Logic animation không bao giờ chạm đến JS thread. Điều này có nghĩa là animation vẫn ở 60fps ngay cả khi JS thread đang bão hòa hoàn toàn.

Reanimated là lựa chọn rõ ràng cho gesture-driven animations — khi pan gesture cần di chuyển một view, hoặc swipe cần kích hoạt một transition. Những thứ này cần phản hồi trong cùng frame với touch event, điều đòi hỏi UI thread execution.

Dùng built-in Animated API cho animation đơn giản, không tương tác — loading spinner, fade-in khi mount, progress bar không phản hồi touch. Những cái này dễ setup hơn và code clean hơn cho các trường hợp cơ bản.

Dùng Reanimated khi: animation phản hồi gesture, bạn cần animate properties không được native driver hỗ trợ, bạn cần interpolation phức tạp synchronized với scroll position, hoặc khi bạn đã profile và thấy Animated API bị drop frame."

---

**Q3 (Mid): "How does the Hermes engine improve startup time?"**

**Model Answer:**
"Before Hermes, React Native apps shipped JavaScript source files and used JavaScriptCore — the same engine Safari uses — to parse and compile them at startup. JIT compilation happens at runtime, so every launch required parsing the JS bundle from scratch.

Hermes precompiles JavaScript to bytecode at build time, before the app is distributed. Instead of shipping a text file of JavaScript, the app ships a binary bytecode file. At startup, Hermes just loads and executes the bytecode directly — no parsing, no compilation.

The practical result is faster Time to Interactive. For large apps, the JS bundle can be megabytes of code to parse. Moving that parsing work offline — to the build step — means the app reaches a usable state significantly faster, which matters especially on lower-end Android devices with slower CPUs.

Hermes also has a smaller memory footprint than JSC and V8. It was specifically designed for constrained mobile environments rather than desktop browser performance.

Since React Native 0.70, Hermes is the default engine for both iOS and Android. If you're on an older project that's still using JSC, switching to Hermes is one of the higher-leverage performance improvements you can make with minimal code changes — it's largely a config flag in your Gradle and Podfile settings."

**Trả lời (Tiếng Việt):**
"Trước Hermes, React Native app ship file JavaScript source và dùng JavaScriptCore — engine mà Safari dùng — để parse và compile chúng lúc startup. JIT compilation xảy ra lúc runtime, vì vậy mỗi lần launch cần parse lại JS bundle từ đầu.

Hermes pre-compile JavaScript thành bytecode lúc build time, trước khi app được phân phối. Thay vì ship file text JavaScript, app ship file bytecode nhị phân. Lúc startup, Hermes chỉ cần load và execute bytecode trực tiếp — không parse, không compile.

Kết quả thực tế là Time to Interactive nhanh hơn. Với app lớn, JS bundle có thể là hàng megabyte code cần parse. Chuyển việc parse đó sang offline — sang bước build — có nghĩa là app đạt được trạng thái có thể dùng được nhanh hơn đáng kể, điều này đặc biệt quan trọng trên các thiết bị Android cấp thấp với CPU chậm hơn.

Hermes cũng có memory footprint nhỏ hơn JSC và V8. Nó được thiết kế đặc biệt cho môi trường mobile hạn chế chứ không phải performance desktop browser.

Từ React Native 0.70, Hermes là engine mặc định cho cả iOS và Android. Nếu bạn đang ở project cũ vẫn dùng JSC, chuyển sang Hermes là một trong những cải thiện performance có đòn bẩy cao nhất bạn có thể làm với thay đổi code tối thiểu — về cơ bản là một config flag trong settings Gradle và Podfile của bạn."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "A user reports a crash on Android but not iOS. How do you debug it?"**

**Model Answer:**
"First, if it's a production crash, I'd check the crash logs. In Expo projects that means checking Sentry or Expo's crash reporting. In a bare workflow app, I'd look at the device's logcat output — `adb logcat` filtered to my app package name.

If I can reproduce it locally, I'd open Android Studio's Logcat window while running on an Android emulator or device. This gives me the full native stack trace, which for native crashes is way more informative than the JavaScript stack.

For JavaScript errors, I'd shake the device to open the Dev Menu and enable remote debugging, or use Flipper's Hermes Debugger to set breakpoints. I'd try to trigger the exact user flow that causes the crash.

Common Android-specific issues I'd look for: permissions that are required on Android but not iOS — like READ_EXTERNAL_STORAGE for file access — missing `google-services.json` config if Firebase is involved, `windowSoftInputMode` behaving differently with keyboard overlap, or Android's back button not being handled and causing unexpected navigation behavior.

I'd also check if any package versions have known Android-specific issues. A quick search of the library's GitHub issues filtered to 'Android' usually surfaces common problems quickly."

**Trả lời (Tiếng Việt):**
"Đầu tiên, nếu đây là production crash, tôi sẽ kiểm tra crash logs. Trong Expo projects điều đó có nghĩa là kiểm tra Sentry hoặc Expo's crash reporting. Trong bare workflow app, tôi sẽ xem logcat output của thiết bị — `adb logcat` filter theo app package name.

Nếu tôi có thể reproduce locally, tôi sẽ mở Android Studio's Logcat window khi chạy trên Android emulator hoặc thiết bị. Cái này cho tôi native stack trace đầy đủ, với native crashes thì thông tin hữu ích hơn nhiều so với JavaScript stack.

Với JavaScript errors, tôi sẽ shake thiết bị để mở Dev Menu và bật remote debugging, hoặc dùng Flipper's Hermes Debugger để đặt breakpoints. Tôi sẽ cố gắng trigger đúng user flow gây ra crash.

Các Android-specific issues phổ biến tôi sẽ tìm: permissions cần trên Android nhưng không cần trên iOS — như READ_EXTERNAL_STORAGE cho file access — thiếu config `google-services.json` nếu có Firebase, `windowSoftInputMode` behave khác với keyboard overlap, hoặc Android's back button không được xử lý gây unexpected navigation behavior.

Tôi cũng kiểm tra xem có package version nào có known Android-specific issues không. Tìm nhanh trên GitHub issues của thư viện lọc theo 'Android' thường phát hiện vấn đề phổ biến nhanh chóng."

---

**Q2 (Mid): "What is Flipper and what specifically can you inspect with it that you can't easily see otherwise?"**

**Model Answer:**
"Flipper is a desktop debugging platform from Meta. You connect it to your running React Native app and get a set of debugging plugins.

What it gives you beyond the standard Dev Menu: first, the Network Inspector. By default, React Native's debugging doesn't capture network requests made by the app. Flipper's network plugin intercepts all `XMLHttpRequest` and `fetch` calls and shows you the full request and response — URL, headers, body, timing. This is the equivalent of the Chrome DevTools Network tab for your phone app.

Second, the Layout Inspector. This shows you the native view hierarchy rendered on screen — every `UIView` on iOS, every `View` on Android, with their exact sizes, positions, and applied styles. You can tap a view to jump to the corresponding component. This is invaluable for debugging layout issues where the visual output doesn't match what you expected.

Third, the Hermes Debugger. For apps using Hermes, you get proper breakpoint debugging — step through code, inspect call stacks, examine variable values. This is much more powerful than `console.log` debugging.

Fourth, Crash Reporter and Logs, which give you a filtered view of device logs without needing to use `adb logcat` or Xcode's console.

That said, Flipper has become less central since React Native 0.73 — it's no longer included by default. Expo's development build with `expo-dev-client` provides a lot of the same capabilities through its own dev tools UI. For network debugging specifically, Reactotron is a solid alternative that doesn't require native integration."

**Trả lời (Tiếng Việt):**
"Flipper là desktop debugging platform từ Meta. Bạn kết nối nó với React Native app đang chạy và nhận được một bộ debugging plugins.

Những gì nó cho bạn vượt ngoài Dev Menu chuẩn: thứ nhất là Network Inspector. Theo mặc định, debugging của React Native không capture network requests do app thực hiện. Network plugin của Flipper intercept tất cả `XMLHttpRequest` và `fetch` calls và cho bạn xem request và response đầy đủ — URL, headers, body, timing. Đây là tương đương của Chrome DevTools Network tab cho phone app của bạn.

Thứ hai là Layout Inspector. Cái này cho bạn xem native view hierarchy được render trên màn hình — mọi `UIView` trên iOS, mọi `View` trên Android, với kích thước, vị trí và styles đã áp dụng của chúng. Bạn có thể tap một view để nhảy đến component tương ứng. Cực kỳ hữu ích khi debug layout issues khi visual output không khớp với kỳ vọng.

Thứ ba là Hermes Debugger. Với app dùng Hermes, bạn có breakpoint debugging đúng nghĩa — step through code, inspect call stacks, xem giá trị biến. Mạnh hơn `console.log` debugging nhiều.

Thứ tư, Crash Reporter và Logs, cho bạn filtered view của device logs mà không cần dùng `adb logcat` hay Xcode's console.

Tuy nhiên, Flipper đã trở nên ít trung tâm hơn từ React Native 0.73 — nó không còn được include mặc định. Expo's development build với `expo-dev-client` cung cấp nhiều khả năng tương tự qua dev tools UI riêng. Với network debugging cụ thể, Reactotron là một alternative tốt không cần native integration."

---

**Q3 (Mid): "How do you debug performance issues in a React Native app — specifically, how do you find what's causing frame drops?"**

**Model Answer:**
"The first tool is the Performance Monitor built into React Native's Dev Menu. It shows two numbers: the UI thread FPS and the JS thread FPS. If only the JS FPS is dropping, the bottleneck is JavaScript computation. If UI FPS drops while JS FPS is fine, it's a native rendering issue. This immediately tells you where to focus.

For JavaScript-level profiling, I'd use the Hermes Sampling Profiler. You start it from the Dev Menu, reproduce the performance issue, then stop it. The profiler downloads a trace file that you open in Chrome DevTools' Performance tab. You see a flame graph of JavaScript execution — which functions are running, how long they take, and what calls them. This is how you find expensive renders or computations.

For native rendering issues, the Layout Inspector in Flipper or the View Hierarchy debugger in Xcode helps spot over-complicated view hierarchies and overdraw — too many overlapping transparent views being composited.

For React-specific re-render issues, the React DevTools Profiler records which components render during an interaction and how long each took. If you see a component re-rendering 50 times during a single scroll, something is creating unstable references — a prop function being created fresh every render, a context value changing unnecessarily.

In practice, I find most performance issues fall into a few categories: unnecessary re-renders from unstable function references in lists, heavy computation happening synchronously on the JS thread during render, or animations driven by the JS thread that should be moved to Reanimated."

**Trả lời (Tiếng Việt):**
"Tool đầu tiên là Performance Monitor có sẵn trong React Native's Dev Menu. Nó hiện hai con số: UI thread FPS và JS thread FPS. Nếu chỉ JS FPS giảm, điểm bottleneck là JavaScript computation. Nếu UI FPS giảm trong khi JS FPS ổn, đó là native rendering issue. Cái này ngay lập tức cho bạn biết cần tập trung ở đâu.

Với JavaScript-level profiling, tôi dùng Hermes Sampling Profiler. Bạn bật nó từ Dev Menu, reproduce performance issue, rồi tắt. Profiler download trace file mà bạn mở trong Chrome DevTools' Performance tab. Bạn thấy flame graph của JavaScript execution — hàm nào đang chạy, mất bao lâu, và cái gì gọi chúng. Đây là cách bạn tìm expensive renders hoặc computations.

Với native rendering issues, Layout Inspector trong Flipper hoặc View Hierarchy debugger trong Xcode giúp phát hiện over-complicated view hierarchies và overdraw — quá nhiều transparent views chồng lên nhau được composite.

Với React-specific re-render issues, React DevTools Profiler record những components nào render trong một interaction và mỗi cái mất bao lâu. Nếu bạn thấy một component re-render 50 lần trong một lần scroll, có gì đó đang tạo unstable references — prop function được tạo mới mỗi render, context value thay đổi không cần thiết.

Trong thực tế, tôi thấy hầu hết performance issues rơi vào một vài category: unnecessary re-renders từ unstable function references trong lists, heavy computation xảy ra synchronously trên JS thread trong render, hoặc animations driven bởi JS thread mà lẽ ra nên chuyển sang Reanimated."

---

**Q4 (Senior): "You have a production React Native app with occasional crashes that users are reporting but you can't reproduce locally. How do you diagnose this?"**

**Model Answer:**
"This is where structured production error tracking pays off. If Sentry or a similar tool is already integrated, crashes in production automatically capture a stack trace, device info, OS version, React Native version, and the user's recent breadcrumbs — what screens they visited, what actions they took before the crash.

If error tracking isn't set up yet, step one is adding it. `@sentry/react-native` with `Sentry.wrap(App)` captures both JavaScript errors and native crashes. For native crashes, Sentry can symbolicate the stack traces using the debug symbols from your builds, turning memory addresses into readable function names.

Once I have a few reports, I'd look for patterns: is it only Android, or iOS too? Specific OS version? Specific device type? A particular user flow — breadcrumbs are helpful here.

For crashes that seem random, memory issues are common suspects in production apps. Memory leaks — event listeners not cleaned up in `useEffect` returns, holding references to unmounted components — accumulate over time and crash when the OS kills the app for using too much RAM. The Hermes Profiler's memory view or Xcode's Instruments can catch these in development once you have a reproduction scenario.

If the crash is happening in native code — a null pointer dereference in a native module, for example — the Sentry native crash report gives a native stack trace. That, combined with the device and OS version, usually narrows it to a specific library version or platform API behavior.

When nothing else works, I'd try to add more telemetry around the suspect area — navigation events, API calls, state changes — to reconstruct what the user was doing, then try to reproduce on a device that matches the reported profile."

**Trả lời (Tiếng Việt):**
"Đây là lúc structured production error tracking phát huy tác dụng. Nếu Sentry hoặc tool tương tự đã được tích hợp, crashes trong production tự động capture stack trace, device info, OS version, React Native version, và user's recent breadcrumbs — những màn hình họ đã visit, những hành động họ thực hiện trước khi crash.

Nếu error tracking chưa được setup, bước một là thêm nó vào. `@sentry/react-native` với `Sentry.wrap(App)` capture cả JavaScript errors và native crashes. Với native crashes, Sentry có thể symbolicate stack traces dùng debug symbols từ builds của bạn, biến memory addresses thành tên hàm đọc được.

Khi đã có một vài reports, tôi sẽ tìm pattern: chỉ Android hay iOS cũng bị? OS version cụ thể? Loại thiết bị cụ thể? User flow cụ thể — breadcrumbs rất hữu ích ở đây.

Với những crashes có vẻ ngẫu nhiên, memory issues là suspect phổ biến trong production apps. Memory leaks — event listeners không được cleanup trong `useEffect` returns, giữ references đến unmounted components — tích lũy theo thời gian và crash khi OS kill app vì dùng quá nhiều RAM. Hermes Profiler's memory view hoặc Xcode's Instruments có thể phát hiện những cái này trong development khi bạn có reproduction scenario.

Nếu crash xảy ra trong native code — null pointer dereference trong native module chẳng hạn — Sentry native crash report cho native stack trace. Kết hợp với device và OS version, thường thu hẹp xuống một thư viện version cụ thể hoặc platform API behavior.

Khi không còn gì khác, tôi sẽ thêm nhiều telemetry hơn xung quanh khu vực bị nghi ngờ — navigation events, API calls, state changes — để reconstruct người dùng đang làm gì, rồi cố reproduce trên thiết bị khớp với profile được báo cáo."

---

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
