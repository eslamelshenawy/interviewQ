# Command Pattern - نمط الأمر

## ما هو Command Pattern / What is Command Pattern

### العربية
**Command Pattern** هو نمط تصميم سلوكي يحول الطلب (Request) إلى كائن مستقل (Object) يحتوي على كل المعلومات عن الطلب. يسمح هذا بتمرير الطلبات كمعاملات، تأخير تنفيذها، وضعها في طابور (Queue)، ودعم العمليات القابلة للتراجع (Undo).

**الفكرة الأساسية:** مثل طلب في مطعم - النادل (Invoker) يأخذ الطلب (Command) ويعطيه للطباخ (Receiver).

### English
**Command Pattern** is a behavioral design pattern that turns a request into a standalone object containing all information about the request. This allows passing requests as method arguments, delaying execution, queuing, and supporting undoable operations.

**Core Idea:** Like an order in a restaurant - waiter (Invoker) takes the order (Command) and gives it to the chef (Receiver).

---

## المشكلة والحل / Problem & Solution

### المشكلة
```javascript
// ❌ ارتباط مباشر بين UI والمنطق
class Button {
  click() {
    // منطق معقد هنا
    document.save();
  }
}
// صعب إضافة Undo/Redo، Queuing، Logging
```

### الحل
```javascript
// ✅ Command Pattern
// Command Interface
class Command {
  execute() { }
  undo() { }
}

// Concrete Commands
class SaveCommand extends Command {
  constructor(document) {
    super();
    this.document = document;
  }

  execute() {
    this.document.save();
  }

  undo() {
    console.log('Undo save');
  }
}

class OpenCommand extends Command {
  constructor(document) {
    super();
    this.document = document;
  }

  execute() {
    this.document.open();
  }

  undo() {
    console.log('Undo open');
  }
}

// Invoker
class Button {
  constructor(command) {
    this.command = command;
  }

  click() {
    this.command.execute();
  }
}

// الاستخدام
const document = { save() { console.log('Saving...'); } };
const saveCommand = new SaveCommand(document);
const button = new Button(saveCommand);
button.click(); // Saving...
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Text Editor with Undo/Redo
```javascript
class Command {
  execute() { }
  undo() { }
}

class WriteCommand extends Command {
  constructor(textEditor, text) {
    super();
    this.editor = textEditor;
    this.text = text;
  }

  execute() {
    this.editor.write(this.text);
  }

  undo() {
    this.editor.delete(this.text.length);
  }
}

class DeleteCommand extends Command {
  constructor(textEditor, length) {
    super();
    this.editor = textEditor;
    this.length = length;
    this.deletedText = '';
  }

  execute() {
    this.deletedText = this.editor.getText().slice(-this.length);
    this.editor.delete(this.length);
  }

  undo() {
    this.editor.write(this.deletedText);
  }
}

class TextEditor {
  constructor() {
    this.content = '';
  }

  write(text) {
    this.content += text;
    console.log(`Content: "${this.content}"`);
  }

  delete(length) {
    this.content = this.content.slice(0, -length);
    console.log(`Content: "${this.content}"`);
  }

  getText() {
    return this.content;
  }
}

class CommandManager {
  constructor() {
    this.history = [];
    this.currentIndex = -1;
  }

  execute(command) {
    command.execute();
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(command);
    this.currentIndex++;
  }

  undo() {
    if (this.currentIndex >= 0) {
      this.history[this.currentIndex].undo();
      this.currentIndex--;
    }
  }

  redo() {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      this.history[this.currentIndex].execute();
    }
  }
}

// الاستخدام
const editor = new TextEditor();
const manager = new CommandManager();

manager.execute(new WriteCommand(editor, 'Hello '));
manager.execute(new WriteCommand(editor, 'World'));
manager.execute(new DeleteCommand(editor, 5));

manager.undo(); // undo delete
manager.undo(); // undo "World"
manager.redo(); // redo "World"
```

### مثال 2: Smart Home
```javascript
class LightOnCommand {
  constructor(light) {
    this.light = light;
  }

  execute() {
    this.light.on();
  }

  undo() {
    this.light.off();
  }
}

class LightOffCommand {
  constructor(light) {
    this.light = light;
  }

  execute() {
    this.light.off();
  }

  undo() {
    this.light.on();
  }
}

class ACOnCommand {
  constructor(ac) {
    this.ac = ac;
  }

  execute() {
    this.ac.on();
    this.ac.setTemperature(22);
  }

  undo() {
    this.ac.off();
  }
}

class Light {
  on() {
    console.log('💡 Light is ON');
  }

  off() {
    console.log('💡 Light is OFF');
  }
}

class AirConditioner {
  on() {
    console.log('❄️ AC is ON');
  }

  off() {
    console.log('❄️ AC is OFF');
  }

  setTemperature(temp) {
    console.log(`❄️ AC temperature set to ${temp}°C`);
  }
}

class RemoteControl {
  constructor() {
    this.commands = [];
    this.history = [];
  }

  setCommand(index, command) {
    this.commands[index] = command;
  }

  pressButton(index) {
    if (this.commands[index]) {
      this.commands[index].execute();
      this.history.push(this.commands[index]);
    }
  }

  pressUndo() {
    if (this.history.length > 0) {
      const command = this.history.pop();
      command.undo();
    }
  }
}

// الاستخدام
const remote = new RemoteControl();
const light = new Light();
const ac = new AirConditioner();

remote.setCommand(0, new LightOnCommand(light));
remote.setCommand(1, new LightOffCommand(light));
remote.setCommand(2, new ACOnCommand(ac));

remote.pressButton(0); // Light ON
remote.pressButton(2); // AC ON
remote.pressUndo(); // Undo AC
```

### مثال 3: Transaction System
```javascript
class Transaction {
  constructor(account, amount) {
    this.account = account;
    this.amount = amount;
  }

  execute() { }
  undo() { }
}

class DepositCommand extends Transaction {
  execute() {
    this.account.balance += this.amount;
    console.log(`Deposited ${this.amount}. Balance: ${this.account.balance}`);
  }

  undo() {
    this.account.balance -= this.amount;
    console.log(`Undo deposit. Balance: ${this.account.balance}`);
  }
}

class WithdrawCommand extends Transaction {
  execute() {
    if (this.account.balance >= this.amount) {
      this.account.balance -= this.amount;
      console.log(`Withdrew ${this.amount}. Balance: ${this.account.balance}`);
      return true;
    }
    console.log('Insufficient funds');
    return false;
  }

  undo() {
    this.account.balance += this.amount;
    console.log(`Undo withdrawal. Balance: ${this.account.balance}`);
  }
}

class TransferCommand {
  constructor(fromAccount, toAccount, amount) {
    this.withdrawCommand = new WithdrawCommand(fromAccount, amount);
    this.depositCommand = new DepositCommand(toAccount, amount);
  }

  execute() {
    if (this.withdrawCommand.execute()) {
      this.depositCommand.execute();
      return true;
    }
    return false;
  }

  undo() {
    this.depositCommand.undo();
    this.withdrawCommand.undo();
  }
}

class Account {
  constructor(name, balance) {
    this.name = name;
    this.balance = balance;
  }
}

// الاستخدام
const account1 = new Account('Ahmed', 1000);
const account2 = new Account('Sara', 500);

const transfer = new TransferCommand(account1, account2, 200);
transfer.execute();
transfer.undo();
```

---

## المزايا والعيوب

### المزايا
- ✅ Single Responsibility - فصل invoker عن receiver
- ✅ Open/Closed - إضافة commands جديدة
- ✅ Undo/Redo operations
- ✅ Queuing operations
- ✅ Logging وTransaction support

### العيوب
- ❌ زيادة عدد الأصناف
- ❌ تعقيد إضافي للعمليات البسيطة

---

## متى تستخدم

استخدم Command عندما:
- ✅ تريد Undo/Redo
- ✅ تريد Queuing operations
- ✅ تريد Logging operations
- ✅ تريد Macro operations
- ✅ تريد فصل Sender عن Receiver

---

## أسئلة المقابلات

**1. ما الفرق بين Command و Strategy؟**
- **Command**: يغلف طلب كامل مع المعاملات
- **Strategy**: يغلف خوارزمية فقط

**2. كيف تنفذ Undo/Redo؟**
احفظ History من Commands واستدعي undo() أو execute()

**3. اذكر أمثلة واقعية؟**
- Ctrl+Z (Undo)
- Task schedulers
- Transaction systems
- GUI button actions
