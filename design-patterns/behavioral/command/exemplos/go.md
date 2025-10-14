# Exemplos Práticos - Command (Go)

## Exemplo 1: Editor de Texto com Undo/Redo

```go
package main

import (
	"fmt"
	"strings"
)

// Command interface
type Command interface {
	Execute()
	Undo()
}

// Receiver - Editor de texto
type TextEditor struct {
	text           strings.Builder
	clipboard      string
	selectionStart int
	selectionEnd   int
}

func NewTextEditor() *TextEditor {
	return &TextEditor{
		text:           strings.Builder{},
		clipboard:      "",
		selectionStart: 0,
		selectionEnd:   0,
	}
}

func (e *TextEditor) Write(text string) {
	e.text.WriteString(text)
}

func (e *TextEditor) ReplaceSelection(newText string) {
	currentText := e.text.String()
	if e.selectionStart >= 0 && e.selectionEnd <= len(currentText) {
		result := currentText[:e.selectionStart] + newText + currentText[e.selectionEnd:]
		e.text.Reset()
		e.text.WriteString(result)
	}
}

func (e *TextEditor) GetSelection() string {
	currentText := e.text.String()
	if e.selectionStart >= 0 && e.selectionEnd <= len(currentText) {
		return currentText[e.selectionStart:e.selectionEnd]
	}
	return ""
}

func (e *TextEditor) GetText() string {
	return e.text.String()
}

func (e *TextEditor) SetText(text string) {
	e.text.Reset()
	e.text.WriteString(text)
}

func (e *TextEditor) SetSelection(start, end int) {
	e.selectionStart = start
	e.selectionEnd = end
}

func (e *TextEditor) GetClipboard() string {
	return e.clipboard
}

func (e *TextEditor) SetClipboard(text string) {
	e.clipboard = text
}

// WriteCommand - Comando de escrever texto
type WriteCommand struct {
	editor       *TextEditor
	textToWrite  string
	previousText string
}

func NewWriteCommand(editor *TextEditor, text string) *WriteCommand {
	return &WriteCommand{
		editor:      editor,
		textToWrite: text,
	}
}

func (c *WriteCommand) Execute() {
	c.previousText = c.editor.GetText()
	c.editor.Write(c.textToWrite)
	fmt.Printf("✏️  Escrevendo: \"%s\"\n", c.textToWrite)
}

func (c *WriteCommand) Undo() {
	c.editor.SetText(c.previousText)
	fmt.Printf("↩️  Desfazendo escrita de: \"%s\"\n", c.textToWrite)
}

// CopyCommand - Comando de copiar
type CopyCommand struct {
	editor *TextEditor
}

func NewCopyCommand(editor *TextEditor) *CopyCommand {
	return &CopyCommand{editor: editor}
}

func (c *CopyCommand) Execute() {
	selected := c.editor.GetSelection()
	c.editor.SetClipboard(selected)
	fmt.Printf("📋 Copiando: \"%s\"\n", selected)
}

func (c *CopyCommand) Undo() {
	fmt.Println("↩️  Copy não tem undo")
}

// CutCommand - Comando de cortar
type CutCommand struct {
	editor            *TextEditor
	previousText      string
	previousClipboard string
}

func NewCutCommand(editor *TextEditor) *CutCommand {
	return &CutCommand{editor: editor}
}

func (c *CutCommand) Execute() {
	c.previousText = c.editor.GetText()
	c.previousClipboard = c.editor.GetClipboard()
	
	selected := c.editor.GetSelection()
	c.editor.SetClipboard(selected)
	c.editor.ReplaceSelection("")
	
	fmt.Printf("✂️  Cortando: \"%s\"\n", selected)
}

func (c *CutCommand) Undo() {
	c.editor.SetText(c.previousText)
	c.editor.SetClipboard(c.previousClipboard)
	fmt.Println("↩️  Desfazendo corte")
}

// PasteCommand - Comando de colar
type PasteCommand struct {
	editor       *TextEditor
	previousText string
}

func NewPasteCommand(editor *TextEditor) *PasteCommand {
	return &PasteCommand{editor: editor}
}

func (c *PasteCommand) Execute() {
	c.previousText = c.editor.GetText()
	clipboard := c.editor.GetClipboard()
	c.editor.ReplaceSelection(clipboard)
	fmt.Printf("📄 Colando: \"%s\"\n", clipboard)
}

func (c *PasteCommand) Undo() {
	c.editor.SetText(c.previousText)
	fmt.Println("↩️  Desfazendo colagem")
}

// CommandHistory - gerencia histórico de comandos
type CommandHistory struct {
	history   []Command
	redoStack []Command
}

func NewCommandHistory() *CommandHistory {
	return &CommandHistory{
		history:   make([]Command, 0),
		redoStack: make([]Command, 0),
	}
}

func (h *CommandHistory) Push(cmd Command) {
	h.history = append(h.history, cmd)
	h.redoStack = make([]Command, 0) // Limpa redo ao executar novo comando
}

func (h *CommandHistory) Undo() Command {
	if len(h.history) > 0 {
		lastIndex := len(h.history) - 1
		cmd := h.history[lastIndex]
		h.history = h.history[:lastIndex]
		h.redoStack = append(h.redoStack, cmd)
		return cmd
	}
	return nil
}

func (h *CommandHistory) Redo() Command {
	if len(h.redoStack) > 0 {
		lastIndex := len(h.redoStack) - 1
		cmd := h.redoStack[lastIndex]
		h.redoStack = h.redoStack[:lastIndex]
		h.history = append(h.history, cmd)
		return cmd
	}
	return nil
}

func (h *CommandHistory) ShowHistory() {
	fmt.Println("\n📚 Histórico de comandos:")
	if len(h.history) == 0 {
		fmt.Println("   (vazio)")
	} else {
		for i, cmd := range h.history {
			fmt.Printf("   %d. %T\n", i+1, cmd)
		}
	}
}

// EditorApplication - Invoker
type EditorApplication struct {
	editor  *TextEditor
	history *CommandHistory
}

func NewEditorApplication() *EditorApplication {
	return &EditorApplication{
		editor:  NewTextEditor(),
		history: NewCommandHistory(),
	}
}

func (app *EditorApplication) ExecuteCommand(cmd Command) {
	cmd.Execute()
	app.history.Push(cmd)
}

func (app *EditorApplication) Undo() {
	cmd := app.history.Undo()
	if cmd != nil {
		cmd.Undo()
	} else {
		fmt.Println("❌ Nada para desfazer")
	}
}

func (app *EditorApplication) Redo() {
	cmd := app.history.Redo()
	if cmd != nil {
		cmd.Execute()
	} else {
		fmt.Println("❌ Nada para refazer")
	}
}

func (app *EditorApplication) ShowContent() {
	fmt.Printf("\n📄 Conteúdo do editor: \"%s\"\n", app.editor.GetText())
}

func (app *EditorApplication) ShowHistory() {
	app.history.ShowHistory()
}

// Métodos auxiliares
func (app *EditorApplication) Write(text string) {
	app.ExecuteCommand(NewWriteCommand(app.editor, text))
}

func (app *EditorApplication) Copy(start, end int) {
	app.editor.SetSelection(start, end)
	app.ExecuteCommand(NewCopyCommand(app.editor))
}

func (app *EditorApplication) Cut(start, end int) {
	app.editor.SetSelection(start, end)
	app.ExecuteCommand(NewCutCommand(app.editor))
}

func (app *EditorApplication) Paste(position int) {
	app.editor.SetSelection(position, position)
	app.ExecuteCommand(NewPasteCommand(app.editor))
}

// Uso
func main() {
	app := NewEditorApplication()
	
	fmt.Println("=== Editor de Texto com Undo/Redo ===")
	
	// Escreve texto
	app.Write("Hello ")
	app.Write("World")
	app.Write("!")
	app.ShowContent()
	
	// Seleciona e copia "World"
	app.Copy(6, 11)
	
	// Cola no final
	app.Paste(12)
	app.ShowContent()
	
	// Corta " World!"
	app.Cut(5, 12)
	app.ShowContent()
	
	// Mostra histórico
	app.ShowHistory()
	
	// Testa undo
	fmt.Println("\n=== Testando Undo ===")
	app.Undo()
	app.ShowContent()
	
	app.Undo()
	app.ShowContent()
	
	// Testa redo
	fmt.Println("\n=== Testando Redo ===")
	app.Redo()
	app.ShowContent()
	
	app.Redo()
	app.ShowContent()
}
```

## Exemplo 2: Sistema de Smart Home

```go
package main

import "fmt"

// Command interface
type HomeCommand interface {
	Execute()
	Undo()
	GetDescription() string
}

// Receivers - dispositivos da casa
type Light struct {
	location   string
	isOn       bool
	brightness int
}

func NewLight(location string) *Light {
	return &Light{
		location:   location,
		isOn:       false,
		brightness: 0,
	}
}

func (l *Light) On() {
	l.isOn = true
	l.brightness = 100
	fmt.Printf("💡 Luz da %s ligada (100%%)\n", l.location)
}

func (l *Light) Off() {
	l.isOn = false
	l.brightness = 0
	fmt.Printf("🌑 Luz da %s desligada\n", l.location)
}

func (l *Light) Dim(level int) {
	l.isOn = true
	l.brightness = level
	fmt.Printf("🔅 Luz da %s ajustada para %d%%\n", l.location, level)
}

func (l *Light) IsOn() bool {
	return l.isOn
}

func (l *Light) GetBrightness() int {
	return l.brightness
}

// Thermostat
type Thermostat struct {
	temperature int
}

func NewThermostat() *Thermostat {
	return &Thermostat{temperature: 22}
}

func (t *Thermostat) SetTemperature(temp int) {
	t.temperature = temp
	fmt.Printf("🌡️  Termostato ajustado para %d°C\n", temp)
}

func (t *Thermostat) GetTemperature() int {
	return t.temperature
}

// SecuritySystem
type SecuritySystem struct {
	armed bool
}

func NewSecuritySystem() *SecuritySystem {
	return &SecuritySystem{armed: false}
}

func (s *SecuritySystem) Arm() {
	s.armed = true
	fmt.Println("🔒 Sistema de segurança ativado")
}

func (s *SecuritySystem) Disarm() {
	s.armed = false
	fmt.Println("🔓 Sistema de segurança desativado")
}

func (s *SecuritySystem) IsArmed() bool {
	return s.armed
}

// Comandos para luz
type LightOnCommand struct {
	light              *Light
	previousBrightness int
}

func NewLightOnCommand(light *Light) *LightOnCommand {
	return &LightOnCommand{light: light}
}

func (c *LightOnCommand) Execute() {
	c.previousBrightness = c.light.GetBrightness()
	c.light.On()
}

func (c *LightOnCommand) Undo() {
	if c.previousBrightness == 0 {
		c.light.Off()
	} else {
		c.light.Dim(c.previousBrightness)
	}
}

func (c *LightOnCommand) GetDescription() string {
	return "Ligar luz"
}

type LightOffCommand struct {
	light              *Light
	previousBrightness int
}

func NewLightOffCommand(light *Light) *LightOffCommand {
	return &LightOffCommand{light: light}
}

func (c *LightOffCommand) Execute() {
	c.previousBrightness = c.light.GetBrightness()
	c.light.Off()
}

func (c *LightOffCommand) Undo() {
	if c.previousBrightness > 0 {
		c.light.Dim(c.previousBrightness)
	}
}

func (c *LightOffCommand) GetDescription() string {
	return "Desligar luz"
}

// Comando para termostato
type ThermostatCommand struct {
	thermostat          *Thermostat
	newTemperature      int
	previousTemperature int
}

func NewThermostatCommand(thermostat *Thermostat, temperature int) *ThermostatCommand {
	return &ThermostatCommand{
		thermostat:     thermostat,
		newTemperature: temperature,
	}
}

func (c *ThermostatCommand) Execute() {
	c.previousTemperature = c.thermostat.GetTemperature()
	c.thermostat.SetTemperature(c.newTemperature)
}

func (c *ThermostatCommand) Undo() {
	c.thermostat.SetTemperature(c.previousTemperature)
}

func (c *ThermostatCommand) GetDescription() string {
	return fmt.Sprintf("Ajustar temperatura para %d°C", c.newTemperature)
}

// Macro Command - executa múltiplos comandos
type MacroCommand struct {
	commands    []HomeCommand
	description string
}

func NewMacroCommand(description string, commands ...HomeCommand) *MacroCommand {
	return &MacroCommand{
		description: description,
		commands:    commands,
	}
}

func (m *MacroCommand) Execute() {
	fmt.Printf("🎬 Executando macro: %s\n", m.description)
	for _, cmd := range m.commands {
		cmd.Execute()
	}
}

func (m *MacroCommand) Undo() {
	fmt.Printf("⏪ Desfazendo macro: %s\n", m.description)
	// Desfaz em ordem reversa
	for i := len(m.commands) - 1; i >= 0; i-- {
		m.commands[i].Undo()
	}
}

func (m *MacroCommand) GetDescription() string {
	return "Macro: " + m.description
}

// Remote Control (Invoker)
type SmartHomeRemote struct {
	commands map[string]HomeCommand
	history  []HomeCommand
}

func NewSmartHomeRemote() *SmartHomeRemote {
	return &SmartHomeRemote{
		commands: make(map[string]HomeCommand),
		history:  make([]HomeCommand, 0),
	}
}

func (r *SmartHomeRemote) SetCommand(button string, cmd HomeCommand) {
	r.commands[button] = cmd
}

func (r *SmartHomeRemote) PressButton(button string) {
	if cmd, exists := r.commands[button]; exists {
		fmt.Printf("\n▶️  Botão pressionado: %s\n", button)
		cmd.Execute()
		r.history = append(r.history, cmd)
	} else {
		fmt.Printf("❌ Botão não configurado: %s\n", button)
	}
}

func (r *SmartHomeRemote) PressUndo() {
	if len(r.history) > 0 {
		lastIndex := len(r.history) - 1
		cmd := r.history[lastIndex]
		r.history = r.history[:lastIndex]
		fmt.Printf("\n↩️  Desfazendo: %s\n", cmd.GetDescription())
		cmd.Undo()
	} else {
		fmt.Println("\n❌ Nada para desfazer")
	}
}

func (r *SmartHomeRemote) ShowCommands() {
	fmt.Println("\n🎮 Comandos configurados:")
	for button, cmd := range r.commands {
		fmt.Printf("   %s: %s\n", button, cmd.GetDescription())
	}
}

// Comando lambda (closure) para segurança
type LambdaCommand struct {
	executeFunc func()
	undoFunc    func()
	description string
}

func NewLambdaCommand(description string, execute, undo func()) *LambdaCommand {
	return &LambdaCommand{
		executeFunc: execute,
		undoFunc:    undo,
		description: description,
	}
}

func (l *LambdaCommand) Execute() {
	l.executeFunc()
}

func (l *LambdaCommand) Undo() {
	l.undoFunc()
}

func (l *LambdaCommand) GetDescription() string {
	return l.description
}

// Uso
func main() {
	// Cria dispositivos (receivers)
	livingRoomLight := NewLight("sala")
	bedroomLight := NewLight("quarto")
	thermostat := NewThermostat()
	security := NewSecuritySystem()
	
	// Cria comandos
	livingRoomOn := NewLightOnCommand(livingRoomLight)
	livingRoomOff := NewLightOffCommand(livingRoomLight)
	bedroomOn := NewLightOnCommand(bedroomLight)
	bedroomOff := NewLightOffCommand(bedroomLight)
	heatUp := NewThermostatCommand(thermostat, 25)
	coolDown := NewThermostatCommand(thermostat, 18)
	
	// Cria macro commands
	movieMode := NewMacroCommand(
		"Modo Cinema",
		livingRoomOff,
		NewThermostatCommand(thermostat, 20),
	)
	
	sleepMode := NewMacroCommand(
		"Modo Dormir",
		livingRoomOff,
		bedroomOff,
		NewThermostatCommand(thermostat, 18),
		NewLambdaCommand(
			"Ativar segurança",
			func() { security.Arm() },
			func() { security.Disarm() },
		),
	)
	
	// Configura controle remoto
	remote := NewSmartHomeRemote()
	remote.SetCommand("Sala ON", livingRoomOn)
	remote.SetCommand("Sala OFF", livingRoomOff)
	remote.SetCommand("Quarto ON", bedroomOn)
	remote.SetCommand("Quarto OFF", bedroomOff)
	remote.SetCommand("Esquentar", heatUp)
	remote.SetCommand("Esfriar", coolDown)
	remote.SetCommand("Cinema", movieMode)
	remote.SetCommand("Dormir", sleepMode)
	
	remote.ShowCommands()
	
	// Simula uso
	fmt.Println("\n=== Chegando em casa ===")
	remote.PressButton("Sala ON")
	remote.PressButton("Esquentar")
	
	fmt.Println("\n=== Hora do cinema ===")
	remote.PressButton("Cinema")
	
	fmt.Println("\n=== Hora de dormir ===")
	remote.PressButton("Dormir")
	
	fmt.Println("\n=== Testando undo ===")
	remote.PressUndo() // Desfaz modo dormir
	remote.PressUndo() // Desfaz modo cinema
}
```
