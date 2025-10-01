# Adapter

> **Também conhecido como:** Wrapper

## 🎯 Intenção

O Adapter é um padrão de projeto estrutural que permite que objetos com interfaces incompatíveis colaborem entre si. Ele atua como um "tradutor" entre duas interfaces diferentes.

## 🚩 Problema

Imagine que você está criando uma aplicação de monitoramento do mercado de ações. A aplicação baixa dados de múltiplas fontes em formato XML e exibe gráficos bonitos para o usuário.

Em determinado momento, você decide melhorar a aplicação integrando uma biblioteca de análise inteligente de terceiros. **Mas há um problema**: a biblioteca só funciona com dados em formato JSON.

### Dilemas comuns:
- **Interface incompatível**: Sua aplicação usa XML, a biblioteca usa JSON
- **Não pode modificar a biblioteca**: Código de terceiros ou sem acesso ao fonte
- **Modificar sua aplicação**: Pode quebrar funcionalidades existentes

```java
// Sua aplicação trabalha com XML
XMLData xmlData = getStockDataXML();

// Biblioteca espera JSON
AnalyticsLibrary library = new AnalyticsLibrary();
library.analyze(jsonData); // ❌ Não funciona com XML
```

## ✅ Solução

Você pode criar um **adapter** - um objeto especial que converte a interface de um objeto para que outro objeto possa entendê-lo.

**Como funciona:**
1. O adapter implementa uma interface compatível com um dos objetos existentes
2. O objeto existente pode chamar métodos do adapter com segurança
3. Ao receber uma chamada, o adapter passa a requisição para o segundo objeto no formato esperado

**Vantagens do Adapter:**
- Esconde a complexidade da conversão
- O objeto adaptado não sabe da existência do adapter
- Pode criar adapters bidirecionais

## 🏗️ Estrutura

### Object Adapter (Composição):
```
Client → ClientInterface ← Adapter → Service
                              ↓
                         (wraps/contains)
```

### Class Adapter (Herança):
```
Client → Adapter extends Service implements ClientInterface
```

### Componentes:
- **Client**: Classe com lógica de negócio existente
- **ClientInterface**: Protocolo que outras classes devem seguir
- **Service**: Classe útil (3ª parte/legacy) com interface incompatível
- **Adapter**: Trabalha com client e service, traduzindo chamadas

## 💻 Exemplo Prático

### Cenário: Sistema de Pagamento

```java
// Interface esperada pelo nosso sistema
interface PaymentProcessor {
    void processPayment(double amount, String currency);
    boolean validatePayment(String details);
}

// Nossa implementação interna
class InternalPaymentProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount, String currency) {
        System.out.printf("Processando pagamento interno: %.2f %s%n", amount, currency);
    }
    
    @Override
    public boolean validatePayment(String details) {
        return details != null && !details.isEmpty();
    }
}

// Biblioteca externa de pagamento (interface incompatível)
class ExternalPaymentGateway {
    public void makePayment(int amountInCents, String currencyCode, String metadata) {
        System.out.printf("Gateway externo: %d centavos %s [%s]%n", 
                         amountInCents, currencyCode, metadata);
    }
    
    public boolean checkTransaction(String transactionInfo) {
        return transactionInfo.contains("VALID");
    }
}

// Adapter para compatibilizar as interfaces
class PaymentGatewayAdapter implements PaymentProcessor {
    private ExternalPaymentGateway externalGateway;
    
    public PaymentGatewayAdapter(ExternalPaymentGateway gateway) {
        this.externalGateway = gateway;
    }
    
    @Override
    public void processPayment(double amount, String currency) {
        // Converte valores para o formato esperado pela biblioteca externa
        int amountInCents = (int) (amount * 100);
        String metadata = "Processed by adapter";
        
        externalGateway.makePayment(amountInCents, currency, metadata);
    }
    
    @Override
    public boolean validatePayment(String details) {
        // Adapta a validação para o formato da biblioteca externa
        String transactionInfo = "VALID:" + details;
        return externalGateway.checkTransaction(transactionInfo);
    }
}

// Sistema de pagamentos que usa a interface padrão
class PaymentSystem {
    private PaymentProcessor processor;
    
    public PaymentSystem(PaymentProcessor processor) {
        this.processor = processor;
    }
    
    public void processOrder(double amount, String currency, String details) {
        if (processor.validatePayment(details)) {
            processor.processPayment(amount, currency);
            System.out.println("Pagamento processado com sucesso!");
        } else {
            System.out.println("Falha na validação do pagamento");
        }
    }
}

// Uso
public class AdapterExample {
    public static void main(String[] args) {
        // Usando processador interno
        PaymentSystem system1 = new PaymentSystem(new InternalPaymentProcessor());
        system1.processOrder(99.99, "USD", "ORDER123");
        
        System.out.println();
        
        // Usando biblioteca externa através do adapter
        ExternalPaymentGateway externalGateway = new ExternalPaymentGateway();
        PaymentGatewayAdapter adapter = new PaymentGatewayAdapter(externalGateway);
        
        PaymentSystem system2 = new PaymentSystem(adapter);
        system2.processOrder(149.50, "EUR", "ORDER456");
    }
}
```

### Exemplo: Media Player

```java
// Interface que nosso sistema entende
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Player interno básico
class AudioPlayer implements MediaPlayer {
    @Override
    public void play(String audioType, String fileName) {
        if ("mp3".equals(audioType)) {
            System.out.println("Tocando MP3: " + fileName);
        } else {
            System.out.println("Formato não suportado: " + audioType);
        }
    }
}

// Players externos avançados (interfaces incompatíveis)
class Mp4Player {
    public void playMp4(String fileName) {
        System.out.println("Tocando MP4: " + fileName);
    }
}

class VlcPlayer {
    public void playVlc(String fileName) {
        System.out.println("Tocando VLC: " + fileName);
    }
}

// Adapter para players externos
class MediaAdapter implements MediaPlayer {
    private Mp4Player mp4Player;
    private VlcPlayer vlcPlayer;
    
    public MediaAdapter(String audioType) {
        if ("mp4".equals(audioType)) {
            mp4Player = new Mp4Player();
        } else if ("vlc".equals(audioType)) {
            vlcPlayer = new VlcPlayer();
        }
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if ("mp4".equals(audioType)) {
            mp4Player.playMp4(fileName);
        } else if ("vlc".equals(audioType)) {
            vlcPlayer.playVlc(fileName);
        }
    }
}

// Player principal com adapter integrado
class AdvancedMediaPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        if ("mp3".equals(audioType)) {
            System.out.println("Tocando MP3: " + fileName);
        } else if ("mp4".equals(audioType) || "vlc".equals(audioType)) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Formato não suportado: " + audioType);
        }
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Interface incompatível**: Quer usar classe existente com interface diferente
- **Classe de terceiros**: Biblioteca externa com interface não compatível
- **Código legado**: Sistema antigo que precisa se integrar com novo
- **Múltiplas subclasses**: Precisam de funcionalidade comum que não pode ser adicionada à superclasse

### 📝 Exemplos de uso:
- **Integração de APIs**: Adaptar diferentes APIs de pagamento
- **Sistemas legados**: Integrar sistemas antigos com novos
- **Bibliotecas de terceiros**: Usar bibliotecas com interfaces diferentes
- **Conversores de dados**: XML para JSON, diferentes formatos

### ❌ Evite quando:
- **Pode modificar o código fonte**: É mais simples alterar diretamente
- **Interface já é compatível**: Não há necessidade de adaptação
- **Complexidade desnecessária**: Para casos muito simples

## 🚀 Como Implementar

1. **Identifique classes incompatíveis**:
   - Classe de serviço útil (não pode alterar)
   - Classes cliente que se beneficiariam do serviço

2. **Declare interface do cliente** e como clientes se comunicam com o serviço

3. **Crie classe adapter** seguindo a interface do cliente

4. **Adicione campo para referência** ao objeto de serviço

5. **Implemente métodos da interface** no adapter:
   - Delegue trabalho real para o objeto de serviço
   - Trate apenas conversão de interface/dados

6. **Clientes usam adapter** via interface do cliente

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Princípio da Responsabilidade Única**: Separa conversão de interface da lógica
- **Princípio Aberto/Fechado**: Novos adapters sem quebrar código existente
- **Reutilização**: Aproveita código existente incompatível
- **Flexibilidade**: Múltiplos adapters para diferentes serviços

### ❌ Desvantagens:
- **Complexidade aumenta**: Introduz novas interfaces e classes
- **Camada extra**: Pode impactar performance
- **Alternativa simples**: Às vezes é mais fácil alterar a classe de serviço

## 🔗 Diferenças de Outros Padrões

| Padrão | Propósito | Interface | Estrutura |
|--------|-----------|-----------|-----------|
| **Adapter** | Compatibilizar interfaces | Muda completamente | Wrapper de objeto |
| **Decorator** | Adicionar comportamento | Mesma ou estendida | Composição recursiva |
| **Facade** | Simplificar subsistema | Nova interface | Múltiplos objetos |
| **Proxy** | Controlar acesso | Mesma interface | Mesmo objeto |

## 🔗 Relações com Outros Padrões

- **Bridge**: Projetado antecipadamente vs Adapter usado com código existente
- **Decorator**: Interface similar/estendida vs Adapter interface diferente
- **Facade**: Interface para subsistema vs Adapter para objeto único
- **Proxy**: Mesma interface vs Adapter interface diferente

## 📚 Conceitos-Chave para Lembrar

1. **Tradução de interfaces**: Essência do padrão
2. **Object vs Class Adapter**: Composição vs herança
3. **Transparência**: Cliente não sabe sobre o adapter
4. **Bidirecionais**: Adapters podem converter nos dois sentidos
5. **Legacy integration**: Casos de uso mais comuns

## 🔍 Analogia do Mundo Real

**Adaptador de tomada de viagem**: Quando você viaja dos EUA para a Europa, precisa de um adaptador para conectar seu laptop. O adaptador tem soquete americano de um lado e plugue europeu do outro, permitindo que dispositivos incompatíveis funcionem juntos.

## ⚠️ Considerações Importantes

### Tipos de Adapter:
- **Object Adapter**: Usa composição (mais flexível)
- **Class Adapter**: Usa herança múltipla (limitado a linguagens que suportam)

### Padrões relacionados:
- **Two-way Adapter**: Conversão bidirecional
- **Pluggable Adapter**: Interface parametrizável
- **Default Adapter**: Implementação padrão vazia

---

> **💡 Dica de Estudo:** Adapter é como um "tradutor" entre dois objetos que falam "idiomas" diferentes. Use quando precisar integrar código existente que não pode ser modificado.

> **📖 Referência:** [Refactoring Guru - Adapter](https://refactoring.guru/design-patterns/adapter)
