# 🖥️ ConfigNG-GUI: Go + WebView Bash Frontend

This project provides a native GUI for Bash commands using Go and [`webview_go`](https://github.com/webview/webview). It opens a simple window with a dropdown menu to run safe predefined commands like `echo`, `date`, and `uptime`.

---

## ✅ Features

- HTML UI embedded in native window
- Predefined command execution (safe, limited)
- Simple dropdown + text input
- Native binary via Go
- No external web server required

---

## 🔧 Requirements

- Go 1.18+  
- Git  
- GCC or Clang for native linking  

On Debian/Ubuntu/ARM (like Khadas VIM3):

```bash
sudo apt install libwebkit2gtk-4.0-dev libgtk-3-dev libglib2.0-dev
```

---

## 🚀 Step-by-Step Setup

### 📁 Step 1: Initialize Your Project Folder

```bash
mkdir configng-gui
cd configng-gui
go mod init configng-gui
```

---

### 📦 Step 2: Get the WebView Dependency

```bash
go get github.com/webview/webview_go@latest
```

This adds it to `go.mod` and fetches it.

---

### 📝 Step 3: Create `main.go`

Create and paste the following code:

```go
package main

import (
	"bytes"
	"os/exec"
	"strings"

	webview "github.com/webview/webview_go"
)

func main() {
	w := webview.New(true)
	defer w.Destroy()
	w.SetTitle("ConfigNG Command GUI")
	w.SetSize(800, 600, webview.HintNone)

	w.Bind("runCommand", func(cmdName, arg string) string {
		cmdName = strings.TrimSpace(cmdName)
		arg = strings.TrimSpace(arg)

		var cmd *exec.Cmd
		switch cmdName {
		case "echo":
			cmd = exec.Command("echo", arg)
		case "date":
			cmd = exec.Command("date")
		case "uptime":
			cmd = exec.Command("uptime")
		default:
			return "Unknown command"
		}

		var out bytes.Buffer
		cmd.Stdout = &out
		err := cmd.Run()
		if err != nil {
			return "Error: " + err.Error()
		}
		return out.String()
	})

	const html = `
	<html>
		<body style="background:#111;color:#eee;font-family:sans-serif;padding:2em;">
			<h1>Run a command</h1>
			<select id="cmd">
				<option value="echo">echo</option>
				<option value="date">date</option>
				<option value="uptime">uptime</option>
			</select>
			<input id="arg" placeholder="optional argument" />
			<button onclick="run()">Run</button>
			<pre id="out" style="background:#222;padding:1em;margin-top:1em;"></pre>
			<script>
				function run() {
					const cmd = document.getElementById('cmd').value;
					const arg = document.getElementById('arg').value;
					window.runCommand(cmd, arg).then(out => {
						document.getElementById('out').textContent = out;
					});
				}
			</script>
		</body>
	</html>`
	w.SetHtml(html)
	w.Run()
}
```

---

### ⚙️ Step 4: Build or Run the App

To run immediately:

```bash
go run main.go
```

To build a binary:

```bash
go build -o configng-gui
./configng-gui
```

---

## 📁 Final Folder Layout

```text
configng-gui/
├── go.mod
├── go.sum
├── main.go
└── configng-gui   # (only after build)
