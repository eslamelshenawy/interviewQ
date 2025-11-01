# Facade Pattern - نمط الواجهة

## ما هو Facade Pattern / What is Facade Pattern

### العربية
**Facade Pattern** هو نمط تصميم هيكلي يوفر واجهة مبسطة (Simplified Interface) لنظام معقد من الأصناف أو المكتبات. يخفي التعقيد ويوفر واجهة سهلة الاستخدام.

**الفكرة الأساسية:** مثل واجهة المتجر - لا تحتاج معرفة كل ما يحدث خلف الكواليس، فقط تستخدم الواجهة البسيطة.

### English
**Facade Pattern** is a structural design pattern that provides a simplified interface to a complex system of classes or libraries. It hides complexity and provides an easy-to-use interface.

**Core Idea:** Like a store front - you don't need to know everything happening behind the scenes, just use the simple interface.

---

## المشكلة والحل / Problem & Solution

### المشكلة
```javascript
// ❌ نظام معقد مع أصناف كثيرة
class VideoFile { }
class AudioMixer { }
class BitrateReader { }
class Codec { }

// العميل يحتاج التعامل مع كل هذا
const file = new VideoFile('video.mp4');
const codec = new Codec();
const bitrate = new BitrateReader();
const mixer = new AudioMixer();
// ... عمليات معقدة
```

### الحل
```javascript
// ✅ Facade يبسط كل شيء
class VideoConverter {
  convert(filename, format) {
    const file = new VideoFile(filename);
    const codec = new Codec();
    const bitrate = new BitrateReader();
    const mixer = new AudioMixer();

    // تنفيذ كل العمليات المعقدة
    return `Converted ${filename} to ${format}`;
  }
}

// الاستخدام بسيط جداً
const converter = new VideoConverter();
converter.convert('video.mp4', 'avi');
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Home Theater Facade
```javascript
class TV {
  on() { console.log('TV is ON'); }
  off() { console.log('TV is OFF'); }
}

class SoundSystem {
  on() { console.log('Sound ON'); }
  setVolume(level) { console.log(`Volume: ${level}`); }
}

class DVDPlayer {
  on() { console.log('DVD Player ON'); }
  play(movie) { console.log(`Playing: ${movie}`); }
}

class Lights {
  dim() { console.log('Lights dimmed'); }
  on() { console.log('Lights ON'); }
}

// Facade
class HomeTheaterFacade {
  constructor() {
    this.tv = new TV();
    this.sound = new SoundSystem();
    this.dvd = new DVDPlayer();
    this.lights = new Lights();
  }

  watchMovie(movie) {
    console.log('Get ready to watch movie...');
    this.lights.dim();
    this.tv.on();
    this.sound.on();
    this.sound.setVolume(10);
    this.dvd.on();
    this.dvd.play(movie);
  }

  endMovie() {
    console.log('Shutting down...');
    this.dvd.off();
    this.tv.off();
    this.sound.off();
    this.lights.on();
  }
}

// الاستخدام
const homeTheater = new HomeTheaterFacade();
homeTheater.watchMovie('Inception');
```

### مثال 2: E-commerce Checkout Facade
```javascript
class Inventory {
  checkStock(productId) {
    return true;
  }
}

class PaymentProcessor {
  processPayment(amount, method) {
    return { success: true, transactionId: '123' };
  }
}

class ShippingService {
  calculateShipping(address) {
    return 10;
  }
  scheduleShipment(order) {
    return 'Shipped';
  }
}

class NotificationService {
  sendEmail(user, message) {
    console.log(`Email sent to ${user}: ${message}`);
  }
}

// Facade
class CheckoutFacade {
  constructor() {
    this.inventory = new Inventory();
    this.payment = new PaymentProcessor();
    this.shipping = new ShippingService();
    this.notification = new NotificationService();
  }

  checkout(order) {
    // 1. Check stock
    if (!this.inventory.checkStock(order.productId)) {
      return { success: false, message: 'Out of stock' };
    }

    // 2. Calculate shipping
    const shippingCost = this.shipping.calculateShipping(order.address);
    const total = order.amount + shippingCost;

    // 3. Process payment
    const payment = this.payment.processPayment(total, order.paymentMethod);
    if (!payment.success) {
      return { success: false, message: 'Payment failed' };
    }

    // 4. Schedule shipment
    this.shipping.scheduleShipment(order);

    // 5. Send confirmation
    this.notification.sendEmail(order.user, 'Order confirmed!');

    return { success: true, orderId: payment.transactionId };
  }
}

// الاستخدام
const checkout = new CheckoutFacade();
checkout.checkout({
  productId: '123',
  amount: 100,
  address: '123 Main St',
  paymentMethod: 'credit_card',
  user: 'ahmed@example.com'
});
```

---

## المزايا والعيوب

### المزايا
- ✅ يبسط واجهة نظام معقد
- ✅ يفصل العميل عن الأنظمة الفرعية
- ✅ سهولة الاستخدام
- ✅ يقلل الاعتماديات

### العيوب
- ❌ قد يصبح Facade نفسه معقداً
- ❌ قد يصبح "God Object"

---

## متى تستخدم

استخدم Facade عندما:
- ✅ نظام معقد يحتاج واجهة مبسطة
- ✅ تريد فصل الطبقات
- ✅ العميل لا يحتاج كل الوظائف
- ✅ تريد تبسيط مكتبة معقدة

---

## أسئلة المقابلات

**1. ما الفرق بين Facade و Adapter؟**
- **Facade**: يبسط واجهة معقدة
- **Adapter**: يحول واجهة غير متوافقة

**2. هل Facade يمنع الوصول المباشر للأنظمة الفرعية؟**
لا، العملاء لا يزال بإمكانهم الوصول مباشرة إذا احتاجوا.

**3. اذكر أمثلة واقعية؟**
- jQuery (facade لـ DOM API)
- Express.js (facade لـ HTTP server)
- Axios (facade لـ XMLHttpRequest)
