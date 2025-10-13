# Mediator

## 🎯 Intenção

O Mediator é um padrão de projeto comportamental que permite reduzir dependências caóticas entre objetos. O padrão restringe comunicações diretas entre objetos e os força a colaborar apenas através de um objeto mediador. Ele centraliza a lógica de comunicação em um único lugar, tornando o sistema mais fácil de entender e manter.

## 🚩 Problema

Imagine que você tem um diálogo para criar e editar perfis de clientes. Ele consiste em vários controles de formulário como campos de texto, checkboxes, botões, etc.

### Resultado problemático:
```java
// Elementos fortemente acoplados
class LoginCheckbox {
    private UsernameField usernameField;
    private PasswordField passwordField;
    private RegisterFields registerFields;
    private SubmitButton submitButton;
    
    public void onCheck() {
        if (isChecked()) {
            // Mostra campos de login
            usernameField.show();
            passwordField.show();
            
            // Esconde campos de registro
            registerFields.hide();
            
            // Atualiza botão
            submitButton.setText("Login");
        } else {
            // Inverte tudo...
            usernameField.hide();
            passwordField.hide();
            registerFields.show();
            submitButton.setText("Register");
        }
    }
}

class SubmitButton {
    private UsernameField usernameField;
    private PasswordField passwordField;
    private EmailField emailField;
    private LoginCheckbox loginCheckbox;
    
    public void onClick() {
        // Valida baseado no estado do checkbox
        if (loginCheckbox.isChecked()) {
            if (!usernameField.isValid()) {
                usernameField.showError("Invalid username");
            }
            if (!passwordField.isValid()) {
                passwordField.showError("Invalid password");
            }
        } else {
            // Valida campos de registro...
            if (!emailField.isValid()) {
                emailField.showError("Invalid email");
            }
            // Mais validações...
        }
    }
}
```

**Problemas:**
- **Acoplamento alto**: Cada elemento conhece muitos outros
- **Difícil reutilização**: Não dá para usar checkbox em outro formulário
- **Manutenção complexa**: Mudança em um elemento afeta vários outros
- **Responsabilidades misturadas**: Lógica de UI espalhada
- **Testes difíceis**: Precisa instanciar muitos objetos
- **Código duplicado**: Lógica de coordenação repetida

## ✅ Solução

O padrão Mediator sugere que você deve cessar toda comunicação direta entre componentes que deseja tornar independentes. Em vez disso, esses componentes devem colaborar indiretamente, chamando um objeto mediador especial que redireciona as chamadas para componentes apropriados.

### Características-chave:
- **Centralização**: Toda comunicação passa pelo mediador
- **Desacoplamento**: Componentes não conhecem uns aos outros
- **Single point**: Um lugar para lógica de coordenação
- **Reusabilidade**: Componentes podem ser reutilizados
- **Testabilidade**: Mais fácil testar isoladamente
- **Manutenibilidade**: Mudanças localizadas no mediador

```java
// Interface Mediator
interface DialogMediator {
    void notify(Component sender, String event);
}

// Componente desacoplado
class LoginCheckbox extends Component {
    public void check() {
        // Apenas notifica o mediador
        mediator.notify(this, "check");
    }
}

class SubmitButton extends Component {
    public void click() {
        // Apenas notifica o mediador
        mediator.notify(this, "click");
    }
}

// Mediador coordena tudo
class AuthDialog implements DialogMediator {
    private LoginCheckbox loginCheckbox;
    private UsernameField usernameField;
    private PasswordField passwordField;
    private SubmitButton submitButton;
    
    @Override
    public void notify(Component sender, String event) {
        if (sender == loginCheckbox && event.equals("check")) {
            if (loginCheckbox.isChecked()) {
                // Mostra campos de login
                usernameField.show();
                passwordField.show();
                submitButton.setText("Login");
            } else {
                // Mostra campos de registro
                // ...
            }
        }
        
        if (sender == submitButton && event.equals("click")) {
            // Valida e processa
            validateAndSubmit();
        }
    }
}
```

## 🏗️ Estrutura

```
Component ----→ Mediator (interface) ←---- ConcreteMediator
   ↓               ↓                              ↓
notify()      notify(sender, event)    [conhece todos componentes]
[referência]                           [coordena interações]
```

### Componentes:
- **Mediator**: Interface que declara método de comunicação
- **ConcreteMediator**: Implementa lógica de coordenação entre componentes
- **Component**: Classe base com referência ao mediador
- **ConcreteComponent**: Componentes específicos que notificam mediador

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Chat com Salas

```java
// Interface Mediator
interface ChatRoomMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
    void removeUser(User user);
}

// Classe base Component
abstract class User {
    protected String name;
    protected ChatRoomMediator mediator;
    protected boolean isOnline;
    
    public User(String name, ChatRoomMediator mediator) {
        this.name = name;
        this.mediator = mediator;
        this.isOnline = true;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message, String senderName);
    
    public String getName() { return name; }
    public boolean isOnline() { return isOnline; }
    public void setOnline(boolean online) { this.isOnline = online; }
}

// Componente concreto
class RegularUser extends User {
    public RegularUser(String name, ChatRoomMediator mediator) {
        super(name, mediator);
    }
    
    @Override
    public void send(String message) {
        System.out.println(name + " enviou: " + message);
        mediator.sendMessage(message, this);
    }
    
    @Override
    public void receive(String message, String senderName) {
        System.out.println(name + " recebeu de " + senderName + ": " + message);
    }
}

// Componente concreto especial
class ModeratorUser extends User {
    public ModeratorUser(String name, ChatRoomMediator mediator) {
        super(name, mediator);
    }
    
    @Override
    public void send(String message) {
        System.out.println("👮 [MOD] " + name + " anunciou: " + message);
        mediator.sendMessage("[MODERADOR] " + message, this);
    }
    
    @Override
    public void receive(String message, String senderName) {
        System.out.println("👮 [MOD] " + name + " recebeu de " + senderName + ": " + message);
    }
    
    public void kickUser(User user) {
        System.out.println("👮 [MOD] " + name + " removeu " + user.getName());
        mediator.removeUser(user);
    }
}

// Mediador concreto
class ChatRoom implements ChatRoomMediator {
    private String roomName;
    private List<User> users;
    private List<String> messageHistory;
    
    public ChatRoom(String roomName) {
        this.roomName = roomName;
        this.users = new ArrayList<>();
        this.messageHistory = new ArrayList<>();
    }
    
    @Override
    public void addUser(User user) {
        users.add(user);
        System.out.println("✅ " + user.getName() + " entrou na sala: " + roomName);
        
        // Notifica todos sobre novo usuário
        String announcement = ">>> " + user.getName() + " entrou na sala";
        for (User u : users) {
            if (!u.equals(user)) {
                u.receive(announcement, "Sistema");
            }
        }
    }
    
    @Override
    public void removeUser(User user) {
        users.remove(user);
        user.setOnline(false);
        System.out.println("❌ " + user.getName() + " saiu da sala: " + roomName);
        
        // Notifica todos sobre saída
        String announcement = "<<< " + user.getName() + " saiu da sala";
        for (User u : users) {
            u.receive(announcement, "Sistema");
        }
    }
    
    @Override
    public void sendMessage(String message, User sender) {
        String fullMessage = sender.getName() + ": " + message;
        messageHistory.add(fullMessage);
        
        // Envia para todos exceto o remetente
        for (User user : users) {
            if (!user.equals(sender) && user.isOnline()) {
                user.receive(message, sender.getName());
            }
        }
    }
    
    public void showHistory() {
        System.out.println("\n=== Histórico da sala " + roomName + " ===");
        for (String msg : messageHistory) {
            System.out.println(msg);
        }
    }
    
    public void showOnlineUsers() {
        System.out.println("\n=== Usuários online na sala " + roomName + " ===");
        for (User user : users) {
            if (user.isOnline()) {
                String type = user instanceof ModeratorUser ? "[MOD]" : "[USER]";
                System.out.println("👤 " + type + " " + user.getName());
            }
        }
    }
}

// Uso
public class ChatExample {
    public static void main(String[] args) {
        // Cria sala
        ChatRoom chatRoom = new ChatRoom("Java Developers");
        
        // Cria usuários
        User alice = new RegularUser("Alice", chatRoom);
        User bob = new RegularUser("Bob", chatRoom);
        User charlie = new RegularUser("Charlie", chatRoom);
        ModeratorUser admin = new ModeratorUser("Admin", chatRoom);
        
        // Adiciona à sala
        System.out.println("=== Usuários entrando na sala ===");
        chatRoom.addUser(alice);
        chatRoom.addUser(bob);
        chatRoom.addUser(charlie);
        chatRoom.addUser(admin);
        
        // Mostra usuários online
        chatRoom.showOnlineUsers();
        
        // Conversação
        System.out.println("\n=== Conversação ===");
        alice.send("Olá pessoal!");
        bob.send("Oi Alice, tudo bem?");
        charlie.send("Alguém sabe sobre o padrão Mediator?");
        admin.send("Bem-vindos! Mantenham o respeito.");
        
        // Moderador remove usuário
        System.out.println("\n=== Ação de moderação ===");
        admin.kickUser(charlie);
        
        // Mais mensagens
        System.out.println("\n=== Mais mensagens ===");
        alice.send("Charlie foi removido!");
        bob.send("O que aconteceu?");
        
        // Histórico
        chatRoom.showHistory();
        
        // Usuários online finais
        chatRoom.showOnlineUsers();
    }
}
```

### Exemplo 2: Torre de Controle de Tráfego Aéreo

```java
// Interface Mediator
interface AirTrafficControl {
    void registerAircraft(Aircraft aircraft);
    void requestLanding(Aircraft aircraft);
    void requestTakeoff(Aircraft aircraft);
    void sendAlert(String message, Aircraft sender);
}

// Classe base Component
abstract class Aircraft {
    protected String flightNumber;
    protected String model;
    protected AirTrafficControl atc;
    protected AircraftStatus status;
    
    public enum AircraftStatus {
        IN_FLIGHT, LANDING, TAKING_OFF, ON_GROUND
    }
    
    public Aircraft(String flightNumber, String model, AirTrafficControl atc) {
        this.flightNumber = flightNumber;
        this.model = model;
        this.atc = atc;
        this.status = AircraftStatus.IN_FLIGHT;
    }
    
    public abstract void land();
    public abstract void takeoff();
    public abstract void emergencyAlert(String message);
    
    public void receiveInstruction(String instruction) {
        System.out.println("  ✈️  " + flightNumber + " recebeu: " + instruction);
    }
    
    public String getFlightNumber() { return flightNumber; }
    public String getModel() { return model; }
    public AircraftStatus getStatus() { return status; }
    public void setStatus(AircraftStatus status) { this.status = status; }
}

// Componente concreto - Avião comercial
class CommercialAircraft extends Aircraft {
    private int passengers;
    
    public CommercialAircraft(String flightNumber, String model, 
                             int passengers, AirTrafficControl atc) {
        super(flightNumber, model, atc);
        this.passengers = passengers;
    }
    
    @Override
    public void land() {
        System.out.println("🛬 " + flightNumber + " (" + model + 
                         ", " + passengers + " passageiros) solicitando pouso");
        atc.requestLanding(this);
    }
    
    @Override
    public void takeoff() {
        System.out.println("🛫 " + flightNumber + " (" + model + 
                         ", " + passengers + " passageiros) solicitando decolagem");
        atc.requestTakeoff(this);
    }
    
    @Override
    public void emergencyAlert(String message) {
        System.out.println("🚨 " + flightNumber + " EMERGÊNCIA: " + message);
        atc.sendAlert(message, this);
    }
    
    public int getPassengers() { return passengers; }
}

// Componente concreto - Avião de carga
class CargoAircraft extends Aircraft {
    private double cargoWeight;
    
    public CargoAircraft(String flightNumber, String model, 
                        double cargoWeight, AirTrafficControl atc) {
        super(flightNumber, model, atc);
        this.cargoWeight = cargoWeight;
    }
    
    @Override
    public void land() {
        System.out.println("📦 " + flightNumber + " (Carga: " + 
                         cargoWeight + "t) solicitando pouso");
        atc.requestLanding(this);
    }
    
    @Override
    public void takeoff() {
        System.out.println("📦 " + flightNumber + " (Carga: " + 
                         cargoWeight + "t) solicitando decolagem");
        atc.requestTakeoff(this);
    }
    
    @Override
    public void emergencyAlert(String message) {
        System.out.println("⚠️ " + flightNumber + " ALERTA: " + message);
        atc.sendAlert(message, this);
    }
}

// Mediador concreto
class ControlTower implements AirTrafficControl {
    private String airportCode;
    private List<Aircraft> registeredAircraft;
    private Queue<Aircraft> landingQueue;
    private Queue<Aircraft> takeoffQueue;
    private boolean runwayAvailable;
    private Aircraft currentUsingRunway;
    
    public ControlTower(String airportCode) {
        this.airportCode = airportCode;
        this.registeredAircraft = new ArrayList<>();
        this.landingQueue = new LinkedList<>();
        this.takeoffQueue = new LinkedList<>();
        this.runwayAvailable = true;
    }
    
    @Override
    public void registerAircraft(Aircraft aircraft) {
        registeredAircraft.add(aircraft);
        System.out.println("📡 Torre " + airportCode + ": " + 
                         aircraft.getFlightNumber() + " registrado");
        aircraft.receiveInstruction("Bem-vindo ao espaço aéreo de " + airportCode);
    }
    
    @Override
    public void requestLanding(Aircraft aircraft) {
        if (runwayAvailable && landingQueue.isEmpty()) {
            approveLanding(aircraft);
        } else {
            landingQueue.offer(aircraft);
            aircraft.receiveInstruction("Aguarde na fila. Posição: " + landingQueue.size());
            System.out.println("  🔵 Torre: " + aircraft.getFlightNumber() + 
                             " na fila de pouso (posição " + landingQueue.size() + ")");
        }
    }
    
    @Override
    public void requestTakeoff(Aircraft aircraft) {
        if (runwayAvailable && takeoffQueue.isEmpty() && landingQueue.isEmpty()) {
            approveTakeoff(aircraft);
        } else {
            takeoffQueue.offer(aircraft);
            aircraft.receiveInstruction("Aguarde liberação. Posição: " + takeoffQueue.size());
            System.out.println("  🔵 Torre: " + aircraft.getFlightNumber() + 
                             " na fila de decolagem (posição " + takeoffQueue.size() + ")");
        }
    }
    
    private void approveLanding(Aircraft aircraft) {
        runwayAvailable = false;
        currentUsingRunway = aircraft;
        aircraft.setStatus(Aircraft.AircraftStatus.LANDING);
        aircraft.receiveInstruction("APROVADO para pouso. Pista liberada!");
        System.out.println("  ✅ Torre: Pouso aprovado para " + aircraft.getFlightNumber());
        
        // Simula tempo de pouso
        simulateLanding(aircraft);
    }
    
    private void approveTakeoff(Aircraft aircraft) {
        runwayAvailable = false;
        currentUsingRunway = aircraft;
        aircraft.setStatus(Aircraft.AircraftStatus.TAKING_OFF);
        aircraft.receiveInstruction("APROVADO para decolagem. Boa viagem!");
        System.out.println("  ✅ Torre: Decolagem aprovada para " + aircraft.getFlightNumber());
        
        // Simula tempo de decolagem
        simulateTakeoff(aircraft);
    }
    
    private void simulateLanding(Aircraft aircraft) {
        // Simula processo de pouso
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {}
        
        aircraft.setStatus(Aircraft.AircraftStatus.ON_GROUND);
        System.out.println("  🏁 " + aircraft.getFlightNumber() + " pousou com segurança");
        freeRunway();
    }
    
    private void simulateTakeoff(Aircraft aircraft) {
        // Simula processo de decolagem
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {}
        
        aircraft.setStatus(Aircraft.AircraftStatus.IN_FLIGHT);
        System.out.println("  🏁 " + aircraft.getFlightNumber() + " decolou com sucesso");
        freeRunway();
    }
    
    private void freeRunway() {
        runwayAvailable = true;
        currentUsingRunway = null;
        processNextRequest();
    }
    
    private void processNextRequest() {
        // Prioridade para pousos
        if (!landingQueue.isEmpty()) {
            Aircraft next = landingQueue.poll();
            System.out.println("  ⏭️  Processando próximo da fila de pouso...");
            approveLanding(next);
        } else if (!takeoffQueue.isEmpty()) {
            Aircraft next = takeoffQueue.poll();
            System.out.println("  ⏭️  Processando próximo da fila de decolagem...");
            approveTakeoff(next);
        }
    }
    
    @Override
    public void sendAlert(String message, Aircraft sender) {
        System.out.println("  🚨 ALERTA DA TORRE: " + message);
        
        // Notifica todas outras aeronaves
        for (Aircraft aircraft : registeredAircraft) {
            if (!aircraft.equals(sender)) {
                aircraft.receiveInstruction("ALERTA: " + sender.getFlightNumber() + 
                                          " reportou - " + message);
            }
        }
        
        // Prioriza emergências
        if (landingQueue.contains(sender)) {
            landingQueue.remove(sender);
            landingQueue.offer(sender); // Move para frente
            System.out.println("  ⚡ Prioridade dada a " + sender.getFlightNumber());
        }
    }
    
    public void showStatus() {
        System.out.println("\n=== Status da Torre " + airportCode + " ===");
        System.out.println("Pista: " + (runwayAvailable ? "✅ Livre" : "❌ Ocupada"));
        if (currentUsingRunway != null) {
            System.out.println("Usando pista: " + currentUsingRunway.getFlightNumber());
        }
        System.out.println("Fila de pouso: " + landingQueue.size() + " aeronaves");
        System.out.println("Fila de decolagem: " + takeoffQueue.size() + " aeronaves");
        System.out.println("Total registrado: " + registeredAircraft.size() + " aeronaves");
    }
}

// Uso
public class AirTrafficExample {
    public static void main(String[] args) throws InterruptedException {
        // Cria torre de controle
        ControlTower tower = new ControlTower("GRU");
        
        System.out.println("=== Aeroporto Internacional GRU ===\n");
        
        // Cria e registra aeronaves
        Aircraft flight1 = new CommercialAircraft("TAM3050", "Boeing 737", 180, tower);
        Aircraft flight2 = new CommercialAircraft("GOL1234", "Airbus A320", 150, tower);
        Aircraft flight3 = new CargoAircraft("CARGO500", "Boeing 747F", 50.5, tower);
        Aircraft flight4 = new CommercialAircraft("LATAM8080", "Boeing 777", 300, tower);
        
        System.out.println("=== Registrando aeronaves ===");
        tower.registerAircraft(flight1);
        tower.registerAircraft(flight2);
        tower.registerAircraft(flight3);
        tower.registerAircraft(flight4);
        
        tower.showStatus();
        
        // Solicita pousos
        System.out.println("\n=== Solicitações de pouso ===");
        flight1.land();
        flight2.land();
        flight3.land();
        
        Thread.sleep(150);
        tower.showStatus();
        
        // Emergência
        System.out.println("\n=== Situação de emergência ===");
        flight4.emergencyAlert("Falha hidráulica - necessito pouso imediato");
        flight4.land();
        
        Thread.sleep(200);
        tower.showStatus();
        
        // Solicita decolagens
        System.out.println("\n=== Solicitações de decolagem ===");
        flight1.takeoff();
        flight2.takeoff();
        
        Thread.sleep(300);
        tower.showStatus();
        
        System.out.println("\n✅ Todas operações concluídas com segurança!");
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Acoplamento alto**: Classes fortemente acopladas
- **Difícil reutilização**: Componentes dependem demais uns dos outros
- **Lógica espalhada**: Coordenação distribuída em muitas classes
- **Muitas subclasses**: Para reutilizar comportamento básico
- **Comunicação complexa**: Muitos objetos comunicando entre si
- **Centralização necessária**: Precisa controlar interações

### 📝 Exemplos de aplicação:
- **UI Frameworks**: Coordenação de controles de formulário
- **Chat systems**: Salas de chat, mensageiros
- **Air traffic control**: Torre de controle
- **MVC**: Controller como mediador
- **Event systems**: Event bus, message broker
- **Game engines**: Sistema de coordenação de entidades

### ❌ Evite quando:
- **Poucos componentes**: Comunicação simples é suficiente
- **Componentes independentes**: Não precisam coordenação
- **God object risk**: Mediador pode ficar muito complexo

## 🚀 Como Implementar

1. **Identifique grupo de classes** fortemente acopladas

2. **Declare interface Mediator** com método de notificação

3. **Implemente mediador concreto** com referências aos componentes

4. **Faça componentes guardarem** referência ao mediador

5. **Mude código dos componentes** para chamar mediador ao invés de outros componentes

6. **Extraia lógica de coordenação** para o mediador

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Single Responsibility**: Extrai comunicação para um lugar
- **Open/Closed**: Novos mediadores sem mudar componentes
- **Desacoplamento**: Componentes independentes
- **Reutilização**: Componentes mais fáceis de reutilizar
- **Centralização**: Lógica de coordenação em um lugar

### ❌ Desvantagens:
- **God Object**: Mediador pode ficar muito complexo
- **Ponto único de falha**: Tudo depende do mediador
- **Performance**: Indireção adicional

## 🔗 Diferenças de Outros Padrões

| Padrão | Comunicação | Conhecimento | Propósito |
|--------|-------------|--------------|-----------|
| **Mediator** | Indireta via mediador | Mediador conhece todos | Reduzir acoplamento |
| **Observer** | Pub/Sub | Componentes independentes | Notificar mudanças |
| **Facade** | Interface simplificada | Facade conhece subsistema | Simplificar interface |
| **Chain of Responsibility** | Sequencial em cadeia | Só conhece próximo | Processar requisição |

## 🔗 Relações com Outros Padrões

- **Facade vs Mediator**:
  - Facade: Simplifica interface, subsistema não conhece facade
  - Mediator: Centraliza comunicação, componentes conhecem mediador

- **Observer + Mediator**: Mediador pode usar Observer internamente
  - Mediador = Publisher
  - Componentes = Subscribers

- **Command**: Mediador pode usar Commands para encapsular requisições

- **Chain of Responsibility**: Ambos sobre comunicação, mas:
  - Chain: Passa requisição sequencialmente
  - Mediator: Redireciona para componente apropriado

## 📚 Conceitos-Chave para Lembrar

1. **Centralização**: Toda comunicação passa pelo mediador
2. **Desacoplamento**: Componentes não conhecem uns aos outros
3. **Single point**: Um lugar para lógica de coordenação
4. **Notificação**: Componentes notificam mediador sobre eventos
5. **Redirecionamento**: Mediador decide quem processar
6. **God Object risk**: Cuidado com mediador muito complexo

## 🔍 Analogia do Mundo Real

**Torre de controle de aeroporto**: Pilotos não conversam diretamente entre si para decidir quem pousa primeiro. Toda comunicação passa pela torre de controle, que coordena todas as aeronaves. Sem a torre, pilotos precisariam estar cientes de cada avião na vizinhança do aeroporto, discutindo prioridades de pouso com dezenas de outros pilotos - isso causaria caos e acidentes. A torre existe apenas para impor restrições na área do terminal, onde o número de atores pode ser esmagador para um piloto.

## ⚠️ Considerações Importantes

### Mediator vs Observer:
```java
// MEDIATOR: Componentes conhecem mediador
class Component {
    private Mediator mediator;
    
    void doSomething() {
        mediator.notify(this, "event");
    }
}

// OBSERVER: Components não se conhecem
class Publisher {
    private List<Subscriber> subscribers;
    
    void notifyAll() {
        for (Subscriber s : subscribers) {
            s.update();
        }
    }
}
```

### Evitando God Object:
```java
// ❌ Mediador muito complexo
class HugeMediator {
    void notify(Component c, String event) {
        if (event.equals("A")) {
            // 100 linhas de lógica
        } else if (event.equals("B")) {
            // 100 linhas de lógica
        }
        // Mais 50 condições...
    }
}

// ✅ Delegue responsabilidades
class BetterMediator {
    private EventHandler eventHandler;
    private ValidationService validator;
    private NotificationService notifier;
    
    void notify(Component c, String event) {
        Event e = eventHandler.parse(event);
        if (validator.validate(e)) {
            notifier.send(e);
        }
    }
}
```

### Mediador com Hierarquia:
```java
// Mediador base
abstract class DialogMediator {
    protected Map<String, Component> components;
    
    public void register(String name, Component component) {
        components.put(name, component);
    }
    
    public abstract void notify(Component sender, String event);
}

// Mediadores especializados
class LoginDialogMediator extends DialogMediator {
    @Override
    public void notify(Component sender, String event) {
        // Lógica específica de login
    }
}

class RegisterDialogMediator extends DialogMediator {
    @Override
    public void notify(Component sender, String event) {
        // Lógica específica de registro
    }
}
```

### Design Guidelines:
- **Interface clara**: Mediador deve ter interface bem definida
- **Responsabilidades limitadas**: Não deixe virar God Object
- **Componentes desacoplados**: Componentes nunca devem se conhecer diretamente
- **Testabilidade**: Mediador facilita testes isolados
- **Combine com outros padrões**: Observer, Command, Strategy

---

> **💡 Dica de Estudo:** Mediator é como um maestro de orquestra - os músicos (componentes) não conversam entre si durante a apresentação, todos olham para o maestro (mediador) que coordena quando cada um deve tocar. Sem o maestro, seria caos com cada músico tentando sincronizar com dezenas de outros!

> **📖 Referência:** [Refactoring Guru - Mediator](https://refactoring.guru/design-patterns/mediator)
