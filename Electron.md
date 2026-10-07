ЛЕГЕНДА

ADSensor

В AdSensor Electron использовали для небольшого desktop-инструмента для команды разработки и аналитиков. Он позволял локально загружать и проверять выгрузки рекламных данных, смотреть результаты обработки и быстро воспроизводить проблемные кейсы без загрузки данных в основной SaaS. Интерфейс был на React и TypeScript, а через Electron и IPC мы работали с локальными файлами.

Я делал React-интерфейс и интеграцию с Electron через preload и IPC. Реализовал загрузку файлов, отображение статусов обработки и передачу данных между renderer и main process. Сам Electron использовали достаточно точечно, основная разработка была на React и TypeScript.

AVS Consulting

Делал небольшой desktop-инструмент для команды, которая работала с 3D-ассетами для проекта. Это было Electron-приложение на React и TypeScript, где можно было просматривать список моделей, проверять их метаданные и локально загружать или обновлять ассеты. Electron использовали, чтобы работать с локальными файлами и папками, а интерфейс полностью делали на React.

Отвечал в основном за React-интерфейс и интеграцию с Electron. Делал каталог ассетов, загрузку файлов, статусы обработки и IPC-взаимодействие с main process. Еще настраивал сборку и упаковку приложения под Linux.

Главная причина была в работе с локальными файлами. В веб-приложении доступ к файловой системе сильно ограничен, а здесь нужно было напрямую работать с папками и 3D-ассетами на компьютере. Electron позволил оставить привычный React-интерфейс и при этом получить доступ к desktop API.

## Что такое Electron

**Electron** - фреймворк для создания десктопных приложений на JavaScript, TypeScript, HTML и CSS.

Позволяет использовать веб-технологии для разработки приложений под:

- Windows
- macOS
- Linux

Примеры приложений: VS Code, Slack, Discord.

---

## Архитектура

Electron объединяет:

- **Chromium** - отвечает за отображение интерфейса.
- **Node.js** - дает доступ к файловой системе, процессам, ОС и другим API.
- **Electron API** - связывает веб-интерфейс с возможностями ОС.

Основные процессы:

### Main Process

Главный процесс приложения.

Отвечает за:

- создание окон;
- работу с файлами и ОС;
- системные события;
- запуск приложения;
- IPC.

```
import { app, BrowserWindow } from 'electron';

app.whenReady().then(() => {
  const window = new BrowserWindow();

  window.loadFile('index.html');
});
```

### Renderer Process

Процесс, в котором отображается UI.

Обычно здесь находится React-приложение:

```
function App() {
  return <h1>Hello Electron</h1>;
}
```

Каждое окно Electron обычно имеет свой renderer process.

---

## IPC

**IPC (Inter-Process Communication)** - механизм межпроцессного взаимодействия. Electron использует IPC для отправки сериализованных сообщений между **main process** и **renderer process**.

Например, React-интерфейс хочет прочитать файл:

```
Renderer
   ↓ IPC
Main
   ↓
File System
   ↓
Main
   ↓ IPC
Renderer
```

Для этого используются:

- `ipcMain` - обработчик на стороне Main.
- `ipcRenderer` - отправитель со стороны Renderer.
- `preload` - безопасный мост между ними.

---

## Preload

`preload` выполняется перед загрузкой Renderer.

Используется для предоставления Renderer ограниченного набора API.

```
contextBridge.exposeInMainWorld('electron', {
  readFile: () => ipcRenderer.invoke('read-file')
});
```

В React:

```
window.electron.readFile();
```

Главная идея:

```
React
  ↓
Preload
  ↓
IPC
  ↓
Main
```

---

## BrowserWindow

`BrowserWindow` создает окно приложения.

```
const win = new BrowserWindow({
  width: 1200,
  height: 800
});
```

В него можно загрузить HTML:

```
win.loadFile('index.html');
```

или веб-приложение:

```
win.loadURL('http://localhost:3000');
```

---

## Безопасность

Renderer не должен напрямую иметь полный доступ к Node.js и файловой системе.

Обычно используют:

```
contextIsolation: true
nodeIntegration: false
```

А доступ к нужным функциям предоставляют через `preload` + `contextBridge`.

---

## Жизненный цикл

Основные события `app`:

```
app.whenReady()
```

Приложение готово создавать окна.

```
app.on('window-all-closed', () => {});
```

Все окна закрыты.

```
app.quit()
```

Полностью завершить приложение.

---

## Electron + React

Типичная структура:

```
Electron App
├── Main Process
│   └── main.ts
│
├── Preload
│   └── preload.ts
│
└── Renderer
    └── React App
        ├── components
        ├── pages
        └── App.tsx
```

React отвечает за **UI**, Electron Main - за **десктопные возможности и ОС**.

---

## Главное для собеседования

**Electron = Chromium + Node.js + API для взаимодействия с ОС.**

Нужно понимать:

- Main Process
- Renderer Process
- Preload
- IPC
- `BrowserWindow`
- `contextBridge`
- `ipcMain` / `ipcRenderer`
- базовые принципы безопасности
- как React интегрируется с Electron