# Exemplos Práticos - Command (Python)

## Exemplo 1: Editor de Texto com Undo/Redo

```python
from abc import ABC, abstractmethod
from typing import List, Optional

# Command interface
class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass
    
    @abstractmethod
    def undo(self) -> None:
        pass

# Receiver - Editor de texto
class TextEditor:
    def __init__(self):
        self._text: List[str] = []
        self._clipboard: str = ""
        self._selection_start: int = 0
        self._selection_end: int = 0
    
    def write(self, text: str) -> None:
        self._text.append(text)
    
    def replace_selection(self, new_text: str) -> None:
        current_text = self.get_text()
        if 0 <= self._selection_start <= self._selection_end <= len(current_text):
            result = (current_text[:self._selection_start] + 
                     new_text + 
                     current_text[self._selection_end:])
            self.set_text(result)
    
    def get_selection(self) -> str:
        current_text = self.get_text()
        if 0 <= self._selection_start <= self._selection_end <= len(current_text):
            return current_text[self._selection_start:self._selection_end]
        return ""
    
    def get_text(self) -> str:
        return "".join(self._text)
    
    def set_text(self, text: str) -> None:
        self._text = [text]
    
    def set_selection(self, start: int, end: int) -> None:
        self._selection_start = start
        self._selection_end = end
    
    def get_clipboard(self) -> str:
        return self._clipboard
    
    def set_clipboard(self, text: str) -> None:
        self._clipboard = text

# WriteCommand - Comando de escrever texto
class WriteCommand(Command):
    def __init__(self, editor: TextEditor, text: str):
        self._editor = editor
        self._text_to_write = text
        self._previous_text: Optional[str] = None
    
    def execute(self) -> None:
        self._previous_text = self._editor.get_text()
        self._editor.write(self._text_to_write)
        print(f"✏️  Escrevendo: \"{self._text_to_write}\"")
    
    def undo(self) -> None:
        if self._previous_text is not None:
            self._editor.set_text(self._previous_text)
        print(f"↩️  Desfazendo escrita de: \"{self._text_to_write}\"")

# CopyCommand - Comando de copiar
class CopyCommand(Command):
    def __init__(self, editor: TextEditor):
        self._editor = editor
    
    def execute(self) -> None:
        selected = self._editor.get_selection()
        self._editor.set_clipboard(selected)
        print(f"📋 Copiando: \"{selected}\"")
    
    def undo(self) -> None:
        print("↩️  Copy não tem undo")

# CutCommand - Comando de cortar
class CutCommand(Command):
    def __init__(self, editor: TextEditor):
        self._editor = editor
        self._previous_text: Optional[str] = None
        self._previous_clipboard: Optional[str] = None
    
    def execute(self) -> None:
        self._previous_text = self._editor.get_text()
        self._previous_clipboard = self._editor.get_clipboard()
        
        selected = self._editor.get_selection()
        self._editor.set_clipboard(selected)
        self._editor.replace_selection("")
        
        print(f"✂️  Cortando: \"{selected}\"")
    
    def undo(self) -> None:
        if self._previous_text is not None:
            self._editor.set_text(self._previous_text)
        if self._previous_clipboard is not None:
            self._editor.set_clipboard(self._previous_clipboard)
        print("↩️  Desfazendo corte")

# PasteCommand - Comando de colar
class PasteCommand(Command):
    def __init__(self, editor: TextEditor):
        self._editor = editor
        self._previous_text: Optional[str] = None
    
    def execute(self) -> None:
        self._previous_text = self._editor.get_text()
        clipboard = self._editor.get_clipboard()
        self._editor.replace_selection(clipboard)
        print(f"📄 Colando: \"{clipboard}\"")
    
    def undo(self) -> None:
        if self._previous_text is not None:
            self._editor.set_text(self._previous_text)
        print("↩️  Desfazendo colagem")

# CommandHistory - gerencia histórico de comandos
class CommandHistory:
    def __init__(self):
        self._history: List[Command] = []
        self._redo_stack: List[Command] = []
    
    def push(self, command: Command) -> None:
        self._history.append(command)
        self._redo_stack.clear()  # Limpa redo ao executar novo comando
    
    def undo(self) -> Optional[Command]:
        if self._history:
            command = self._history.pop()
            self._redo_stack.append(command)
            return command
        return None
    
    def redo(self) -> Optional[Command]:
        if self._redo_stack:
            command = self._redo_stack.pop()
            self._history.append(command)
            return command
        return None
    
    def show_history(self) -> None:
        print("\n📚 Histórico de comandos:")
        if not self._history:
            print("   (vazio)")
        else:
            for i, cmd in enumerate(self._history, 1):
                print(f"   {i}. {cmd.__class__.__name__}")

# EditorApplication - Invoker
class EditorApplication:
    def __init__(self):
        self._editor = TextEditor()
        self._history = CommandHistory()
    
    def execute_command(self, command: Command) -> None:
        command.execute()
        self._history.push(command)
    
    def undo(self) -> None:
        command = self._history.undo()
        if command:
            command.undo()
        else:
            print("❌ Nada para desfazer")
    
    def redo(self) -> None:
        command = self._history.redo()
        if command:
            command.execute()
        else:
            print("❌ Nada para refazer")
    
    def show_content(self) -> None:
        print(f"\n📄 Conteúdo do editor: \"{self._editor.get_text()}\"")
    
    def show_history(self) -> None:
        self._history.show_history()
    
    # Métodos auxiliares
    def write(self, text: str) -> None:
        self.execute_command(WriteCommand(self._editor, text))
    
    def copy(self, start: int, end: int) -> None:
        self._editor.set_selection(start, end)
        self.execute_command(CopyCommand(self._editor))
    
    def cut(self, start: int, end: int) -> None:
        self._editor.set_selection(start, end)
        self.execute_command(CutCommand(self._editor))
    
    def paste(self, position: int) -> None:
        self._editor.set_selection(position, position)
        self.execute_command(PasteCommand(self._editor))

# Uso
if __name__ == "__main__":
    app = EditorApplication()
    
    print("=== Editor de Texto com Undo/Redo ===")
    
    # Escreve texto
    app.write("Hello ")
    app.write("World")
    app.write("!")
    app.show_content()
    
    # Seleciona e copia "World"
    app.copy(6, 11)
    
    # Cola no final
    app.paste(12)
    app.show_content()
    
    # Corta " World!"
    app.cut(5, 12)
    app.show_content()
    
    # Mostra histórico
    app.show_history()
    
    # Testa undo
    print("\n=== Testando Undo ===")
    app.undo()
    app.show_content()
    
    app.undo()
    app.show_content()
    
    # Testa redo
    print("\n=== Testando Redo ===")
    app.redo()
    app.show_content()
    
    app.redo()
    app.show_content()
```

## Exemplo 2: Sistema de Smart Home

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Callable

# Command interface
class HomeCommand(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass
    
    @abstractmethod
    def undo(self) -> None:
        pass
    
    @abstractmethod
    def get_description(self) -> str:
        pass

# Receivers - dispositivos da casa
class Light:
    def __init__(self, location: str):
        self.location = location
        self._is_on = False
        self._brightness = 0
    
    def on(self) -> None:
        self._is_on = True
        self._brightness = 100
        print(f"💡 Luz da {self.location} ligada (100%)")
    
    def off(self) -> None:
        self._is_on = False
        self._brightness = 0
        print(f"🌑 Luz da {self.location} desligada")
    
    def dim(self, level: int) -> None:
        self._is_on = True
        self._brightness = level
        print(f"🔅 Luz da {self.location} ajustada para {level}%")
    
    def is_on(self) -> bool:
        return self._is_on
    
    def get_brightness(self) -> int:
        return self._brightness

class Thermostat:
    def __init__(self):
        self._temperature = 22
    
    def set_temperature(self, temp: int) -> None:
        self._temperature = temp
        print(f"🌡️  Termostato ajustado para {temp}°C")
    
    def get_temperature(self) -> int:
        return self._temperature

class SecuritySystem:
    def __init__(self):
        self._armed = False
    
    def arm(self) -> None:
        self._armed = True
        print("🔒 Sistema de segurança ativado")
    
    def disarm(self) -> None:
        self._armed = False
        print("🔓 Sistema de segurança desativado")
    
    def is_armed(self) -> bool:
        return self._armed

# Comandos para luz
class LightOnCommand(HomeCommand):
    def __init__(self, light: Light):
        self._light = light
        self._previous_brightness = 0
    
    def execute(self) -> None:
        self._previous_brightness = self._light.get_brightness()
        self._light.on()
    
    def undo(self) -> None:
        if self._previous_brightness == 0:
            self._light.off()
        else:
            self._light.dim(self._previous_brightness)
    
    def get_description(self) -> str:
        return "Ligar luz"

class LightOffCommand(HomeCommand):
    def __init__(self, light: Light):
        self._light = light
        self._previous_brightness = 0
    
    def execute(self) -> None:
        self._previous_brightness = self._light.get_brightness()
        self._light.off()
    
    def undo(self) -> None:
        if self._previous_brightness > 0:
            self._light.dim(self._previous_brightness)
    
    def get_description(self) -> str:
        return "Desligar luz"

# Comando para termostato
class ThermostatCommand(HomeCommand):
    def __init__(self, thermostat: Thermostat, temperature: int):
        self._thermostat = thermostat
        self._new_temperature = temperature
        self._previous_temperature = 0
    
    def execute(self) -> None:
        self._previous_temperature = self._thermostat.get_temperature()
        self._thermostat.set_temperature(self._new_temperature)
    
    def undo(self) -> None:
        self._thermostat.set_temperature(self._previous_temperature)
    
    def get_description(self) -> str:
        return f"Ajustar temperatura para {self._new_temperature}°C"

# Lambda Command - para comandos simples
class LambdaCommand(HomeCommand):
    def __init__(self, description: str, execute_fn: Callable, undo_fn: Callable):
        self._description = description
        self._execute_fn = execute_fn
        self._undo_fn = undo_fn
    
    def execute(self) -> None:
        self._execute_fn()
    
    def undo(self) -> None:
        self._undo_fn()
    
    def get_description(self) -> str:
        return self._description

# Macro Command - executa múltiplos comandos
class MacroCommand(HomeCommand):
    def __init__(self, description: str, *commands: HomeCommand):
        self._description = description
        self._commands = list(commands)
    
    def execute(self) -> None:
        print(f"🎬 Executando macro: {self._description}")
        for command in self._commands:
            command.execute()
    
    def undo(self) -> None:
        print(f"⏪ Desfazendo macro: {self._description}")
        # Desfaz em ordem reversa
        for command in reversed(self._commands):
            command.undo()
    
    def get_description(self) -> str:
        return f"Macro: {self._description}"

# Remote Control (Invoker)
class SmartHomeRemote:
    def __init__(self):
        self._commands: Dict[str, HomeCommand] = {}
        self._history: List[HomeCommand] = []
    
    def set_command(self, button: str, command: HomeCommand) -> None:
        self._commands[button] = command
    
    def press_button(self, button: str) -> None:
        if button in self._commands:
            command = self._commands[button]
            print(f"\n▶️  Botão pressionado: {button}")
            command.execute()
            self._history.append(command)
        else:
            print(f"❌ Botão não configurado: {button}")
    
    def press_undo(self) -> None:
        if self._history:
            command = self._history.pop()
            print(f"\n↩️  Desfazendo: {command.get_description()}")
            command.undo()
        else:
            print("\n❌ Nada para desfazer")
    
    def show_commands(self) -> None:
        print("\n🎮 Comandos configurados:")
        for button, command in self._commands.items():
            print(f"   {button}: {command.get_description()}")

# Uso
if __name__ == "__main__":
    # Cria dispositivos (receivers)
    living_room_light = Light("sala")
    bedroom_light = Light("quarto")
    thermostat = Thermostat()
    security = SecuritySystem()
    
    # Cria comandos
    living_room_on = LightOnCommand(living_room_light)
    living_room_off = LightOffCommand(living_room_light)
    bedroom_on = LightOnCommand(bedroom_light)
    bedroom_off = LightOffCommand(bedroom_light)
    heat_up = ThermostatCommand(thermostat, 25)
    cool_down = ThermostatCommand(thermostat, 18)
    
    # Cria macro commands
    movie_mode = MacroCommand(
        "Modo Cinema",
        living_room_off,
        ThermostatCommand(thermostat, 20)
    )
    
    sleep_mode = MacroCommand(
        "Modo Dormir",
        living_room_off,
        bedroom_off,
        ThermostatCommand(thermostat, 18),
        LambdaCommand(
            "Ativar segurança",
            lambda: security.arm(),
            lambda: security.disarm()
        )
    )
    
    # Configura controle remoto
    remote = SmartHomeRemote()
    remote.set_command("Sala ON", living_room_on)
    remote.set_command("Sala OFF", living_room_off)
    remote.set_command("Quarto ON", bedroom_on)
    remote.set_command("Quarto OFF", bedroom_off)
    remote.set_command("Esquentar", heat_up)
    remote.set_command("Esfriar", cool_down)
    remote.set_command("Cinema", movie_mode)
    remote.set_command("Dormir", sleep_mode)
    
    remote.show_commands()
    
    # Simula uso
    print("\n=== Chegando em casa ===")
    remote.press_button("Sala ON")
    remote.press_button("Esquentar")
    
    print("\n=== Hora do cinema ===")
    remote.press_button("Cinema")
    
    print("\n=== Hora de dormir ===")
    remote.press_button("Dormir")
    
    print("\n=== Testando undo ===")
    remote.press_undo()  # Desfaz modo dormir
    remote.press_undo()  # Desfaz modo cinema
```
