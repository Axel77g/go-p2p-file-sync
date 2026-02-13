# Go Peer-to-Peer File Synchronization

> 🚧 **POC d'un système de synchronisation décentralisée de fichiers**  
> ✅ **Principe** : Découverte P2P, Event Sourcing inspiré de Git, Architecture hexagonal et extensible  
> ⚠️ **État** : Prototype fonctionnel mais non finalisé (contraintes de temps)  
> 📊 [Présentation technique](https://docs.google.com/presentation/d/1kEGpoJS19hbZO0XidA5cflE7A8PdfhAqCd6JQrOz954/edit?usp=sharing)

<!-- Go version (équivalent au badge Node) -->
![Go Version](https://img.shields.io/badge/go-%3E%3D1.18-blue)


---

## 📋 Contexte & Objectif

**Problème adressé :**  
Créer un système de synchronisation décentralisée de fichiers, inspiré de Git, avec découverte automatique des pairs et propagation d'événements.

**Cas d'usage ciblé :**  
- Synchronisation locale sans serveur central
- Traçabilité complète des modifications (event sourcing)
- Architecture distribuée résiliente

**État du projet :**  
⏱️ **Projet académique** développé sous contraintes de temps  
✅ **Ce qui fonctionne** : Découverte P2P (UDP), synchronisation d'événements (TCP), event sourcing  
🚧 **Ce qui manque** : Transfert effectif des fichiers, résolution de conflits, reconstruction du répertoire

---

## 🎯 Réalisations techniques

| Catégorie | Technologies & Concepts |
|-----------|------------------------|
| **Backend Go** | Concurrency (goroutines, channels), interfaces, composition |
| **Architecture** | Event Sourcing, Clean Architecture, Separation of Concerns, Protocol Abstraction |
| **Réseau** | UDP (discovery), TCP (sync) |
| **Systèmes distribués** | P2P networking, Peer discovery, Event synchronization, Eventual consistency |
| **Patterns** | Handler pattern, Strategy pattern, Observer pattern, Iterator, Singleton |
| **Persistence** | JSONL event store, Append-only log |
| **Qualité** | Thread-safety (mutex, atomic), Tests unitaires, Error handling |

---

## 🏗️ Architecture

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                         Application                             │
├──────────────┬──────────────────────┬───────────────────────────┤
│   Handlers   │   Network Layer      │   File System Layer       │
│   (Logic)    │   (Transport)        │   (Events)                │
│              │                      │                           │
│ ┌──────────┐ │ ┌─────────────────┐  │ ┌─────────────────────┐   │
│ │ UDP      │ │ │ ITransportChannel│  │ │ FileWatcher         │  │
│ │Discovery │←┼─│   (Interface)   │  │ │ (Observer)          │   │
│ └──────────┘ │ │  ┌───┐   ┌───┐  │  │ └─────────────────────┘   │
│              │ │  │UDP│   │TCP│  │  │                           │
│ ┌──────────┐ │ │  └───┘   └───┘  │  │ ┌─────────────────────┐   │ 
│ │ TCP      │←┼─│                 │  │ │ EventManager        │   │
│ │Sync      │ │ │ PeerManager     │  │ │ (Singleton)         │   │
│ └──────────┘ │ │ (Thread-safe)   │  │ └─────────┬───────────┘   │
│              │ └─────────────────┘  │           │               │ 
│              │                      │           ▼               │
│              │                      │ ┌─────────────────────┐   │
│              │                      │ │ JSONL Collection    │   │
│              │                      │ │ (Event Sourcing)    │   │
│              │                      │ └─────────────────────┘   │
└──────────────┴──────────────────────┴───────────────────────────┘
```

### Composants clés (implémentés)

#### 1. **Network Layer** - ✅ Fonctionnel
- **`ITransportChannel`** : Interface unifiée pour UDP/TCP
- **`ITransportChannelHandler`** : Pattern pour événements réseau (OnOpen, OnMessage, OnClose)
- **`PeerManager`** : Registre thread-safe avec gestion multi-canaux par pair
- **Abstraction protocole** : Logique métier découplée du transport

#### 2. **Event Sourcing System (inspiré de GIT)** - ✅ Fonctionnel
- **`IFileEventCollection`** : Interface pour collections d'événements
- **`JSONLFileEventCollection`** : Append-only log avec JSONL (inspiré de la blockchain)
- **`FileEventIterator`** : Parcours lazy avec protection concurrence (évite de charger tout l'historique d'event en mémoire)
- **`EventManager`** : Fusion et diffusion d'événements

#### 3. **P2P Discovery** - ✅ Fonctionnel
- **UDP Broadcast** : Découverte automatique des pairs sur réseau local
- **Établissement connexions TCP** : Automatique après découverte

#### 4. **Event Synchronization** - ✅ Partiellement fonctionnel
- **PULL_EVENTS_REQUEST/RESPONSE** : Protocole de synchronisation
- **Fusion d'événements** : Dédoublonnage par hash
- **Streaming** : `SendIterator` pour envoi efficace (sans chargement mémoire complet)

⚠️ **Limitation actuelle** : Seuls les **métadonnées des événements** sont synchronisées, pas le contenu effectif des fichiers.

---

## ⚙️ Fonctionnalités (État actuel)

### ✅ Ce qui fonctionne

| Fonctionnalité | État | Description |
|---------------|------|-------------|
| **Découverte P2P** | ✅ Complet | UDP broadcast, détection automatique des pairs |
| **Connexions TCP** | ✅ Complet | Établissement automatique après découverte |
| **Event Sourcing** | ✅ Complet | Append-only JSONL, historique immuable |
| **File Watcher** | ✅ Complet | Détection temps-réel des modifications |
| **Synchronisation événements** | ⚠️ Partiel | Métadonnées synchronisées, pas le contenu |
| **Fusion d'événements** | ✅ Complet | merge chronologique |
| **Architecture extensible** | ✅ Complet | Interfaces pour compression/encryption |

### 🚧 Ce qui manque (temps insuffisant)

| Fonctionnalité | État | Impact |
|---------------|------|--------|
| **Transfert de fichiers** | ❌ Non implémenté | Les fichiers ne sont pas copiés entre pairs |
| **Reconstruction répertoire** | ❌ Non implémenté | Impossible de recréer l'état à partir des événements |
| **Résolution de conflits** | ❌ Non implémenté | Modifications simultanées non gérées |
| **Compression active** | 🟡 Code prêt | Module implémenté mais non activé |
| **Encryption** | 🟡 Code prêt | Module implémenté mais non activé |


---

## 🔧 Choix techniques & patterns

### 1. **Architecture hexagonale**
```go
// Abstraction transport : logique métier indépendante du protocole
type ITransportChannel interface {
    Send(content []byte) error
    SendIterator(message []byte, iterator shared.Iterator) error
}
```
👉 Permet d'ajouter de nouveaux protocoles sans toucher aux handlers.

### 2. **Handler Pattern**
```go
// Inversion de contrôle pour les événements réseau
type ITransportChannelHandler interface {
    OnOpen(channel ITransportChannel)
    OnMessage(channel ITransportChannel, message TransportMessage) error
    OnClose(channel ITransportChannel)
}
```
👉 **UDPDiscoveryHandler** et **TCPControllerHandler** implémentent des logiques différentes sans duplication.

### 3. **Event Sourcing à la Git**
```go
type FileEvent struct {
    Timestamp  int64  `json:"timestamp"`
    Type       string `json:"type"`        // CREATE, UPDATE, DELETE
    Path       string `json:"path"`
    Hash       string `json:"hash"`
}
```
👉 **Avantages** : historique complet, fusion possible, reconstruction théorique de l'état. 

### 4. **Concurrency Go**
```go
// main.go - 6 goroutines concurrentes
go tcpServer.Listen(&handlers.TCPControllerTransportChannelHandler{})
go udpServer.Listen(&handlers.UDPDiscoveryTransportChannel{})
go discovery.SenderLoop(shared.SOCKET_ID, networkInterfaceManager)
go watcher.Listen()
go func() { /* event broadcast */ }()
```

### 5. **Thread-Safety**
```go
// app/peer_comunication/peer_manager.go
var peersMutex = sync.Mutex{} // Registre global protégé

// app/files/event/jsonl_event_collection.go
activeIterators atomic.Int32 // Empêche écriture pendant lecture
```

---

## 🚀 Mise en route rapide

### Prérequis
- **Go** 1.18+ installé
- Deux machines sur le **même réseau local**
- Ports **UDP/TCP** ouverts (configuration pare-feu)

### Lancement
```bash
# Sur chaque machine
git clone https://github.com/Axel77g/go-peer-to-peer.git
cd go-peer-to-peer
go run main.go
```

### Résultat attendu
1. ✅ Les pairs se **découvrent automatiquement** (logs dans le terminal)
2. ✅ Les connexions TCP s'établissent
3. ✅ Les événements de fichiers sont détectés (création/modification/suppression dans `./shared/`)
4. ✅ Les métadonnées sont synchronisées (visible dans `./events.jsonl`)
5. ⚠️ **Les fichiers eux-mêmes ne sont PAS copiés** (seulement les événements)

---

## 📊 Flux de synchronisation (actuel)


---

## 🔬 Roadmap (si le projet était continué)

| Priorité | Fonctionnalité | Effort | Impact |
|----------|---------------|--------|--------|
| **P0** | **Transfert de fichiers** | 2-3 jours | ⭐⭐⭐ Rend le système utilisable |
| **P0** | **Reconstruction répertoire** | 1-2 jours | ⭐⭐⭐ Exploitation des événements |
| **P1** | **Résolution de conflits** | 3-5 jours | ⭐⭐ CRDTs ou vector clocks |
| **P2** | **Activation compression** | Quelques heures | ⭐ Optimisation bande passante |
| **P2** | **Activation encryption** | Quelques heures | ⭐ Sécurité |
| **P3** | **Interface Web** | 2-3 jours | ⭐ UX/monitoring |
| **P3** | **NAT Traversal** | 5+ jours | ⭐ Internet-wide P2P |

---

## 🧪 Tests

Tests unitaires disponibles :
```bash
go test ./app/compression    # ✅ Complet
go test ./app/encryption     # ✅ Complet
go test ./app/files/event    # ✅ Complet
```

---

## 💡 Philosophie de conception

### Principes appliqués

| Principe | Application concrète |
|----------|---------------------|
| **SOLID** | Interfaces minimales (`ITransportChannel`), handlers découplés |
| **Event-Driven** | Tout changement = événement immuable |
| **Protocol Abstraction** | Logique métier indépendante d'UDP/TCP |
| **Extensibility** | Compression/encryption ajoutables sans refactoring |
| **Concurrency** | Goroutines + channels pour I/O parallèle |

### Design inspiré de Git

```bash
# Git stocke les commits (événements)
git log --oneline
# a1b2c3d Update file.txt (CREATE)
# d4e5f6a Delete old.txt  (DELETE)

# Notre système fait pareil
cat events.jsonl
# {"timestamp":1234,"type":"CREATE","path":"file.txt","hash":"..."}
# {"timestamp":5678,"type":"DELETE","path":"old.txt","hash":"..."}
```

--- 

## 📝 Licence

Projet académique — ESGI 5ème année

---

<p align="center">
  <b>Développé par <a href="https://github.com/Axel77g">Axel77g</a></b><br>
</p>
