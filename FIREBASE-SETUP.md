# 🔥 Configuração do Firebase Realtime Database

## 📋 Passos para Importar a Estrutura

### 1. Acesse o Firebase Console
- Vá para: https://console.firebase.google.com
- Selecione seu projeto: **contaconjunta-marc35**

### 2. Navegue até o Realtime Database
- No menu lateral, clique em **Realtime Database**
- Se ainda não criou, clique em "Criar banco de dados"
- Escolha o modo de teste (você pode ajustar as regras depois)

### 3. Importe a Estrutura Inicial
1. No painel do Realtime Database, clique nos **3 pontinhos** no canto superior direito
2. Selecione **"Importar JSON"**
3. Escolha o arquivo `firebase-database-structure.json`
4. Clique em **Importar**

### 4. Verifique a Estrutura
Após a importação, você deve ver:

```
├─ usuarios_app/
│  ├─ marcelino/
│  │  ├─ nome: "Marcelino"
│  │  ├─ email: "marcelino@email.com"
│  │  └─ senha: "senha123"
│  ├─ luiza/
│  │  ├─ nome: "Luiza"
│  │  ├─ email: "luiza@email.com"
│  │  └─ senha: "senha123"
│  └─ casal/
│     ├─ nome: "Casal"
│     ├─ email: "casal@email.com"
│     └─ senha: "senha123"
└─ transactions/
   ├─ marcelino/
   ├─ luiza/
   └─ casalusuario/
```

## 🔐 Credenciais de Login

### Usuário Marcelino
- **ID:** marcelino
- **Senha:** senha123
- **Perfil:** Usuário Individual (user_a)
- **Vê:** Apenas suas próprias transações

### Usuária Luiza
- **ID:** luiza
- **Senha:** senha123
- **Perfil:** Usuário Individual (user_b)
- **Vê:** Apenas suas próprias transações

### Login Casal - 2 Perfis Disponíveis
- **ID:** casal
- **Senha:** senha123

#### Perfil 1: Admin (Visão Completa)
- **Vê:** Todas as transações (Marcelino + Luiza + Conta Conjunta)
- **Pode criar:** Transações para qualquer um dos 3
- **Ideal para:** Revisão completa das finanças familiares

#### Perfil 2: Conta Conjunta
- **Vê:** Apenas transações da conta conjunta (casalusuario)
- **Pode criar:** Apenas transações conjuntas
- **Ideal para:** Gerenciar despesas compartilhadas do casal

## 🔧 Regras de Segurança (Opcional)

Para produção, configure as regras no Firebase Console:

```json
{
  "rules": {
    "usuarios_app": {
      ".read": "auth != null",
      ".write": false
    },
    "transactions": {
      "$userId": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }
}
```

## ✅ Correções Implementadas

### Problema Anterior
- Transações não eram salvas no banco de dados
- Usava debounce de 2 segundos (perdia transações rápidas)
- Apenas a última transação era sincronizada
- Dados eram salvos no localStorage (não sincronizava entre dispositivos)

### Solução Aplicada
- ✅ Salvamento **imediato** de todas as transações
- ✅ Tracking com `useRef` para evitar duplicatas
- ✅ Transações carregadas do banco são marcadas como já sincronizadas
- ✅ Listener em tempo real também marca como sincronizado
- ✅ Limpeza do tracking ao fazer logout
- ✅ **REMOVIDO localStorage completamente** - 100% Firebase
- ✅ Zustand usado apenas para estado em memória durante a sessão
- ✅ Real-time sync entre múltiplos dispositivos/abas

## 🧪 Testando

### Teste 1: Salvamento no Firebase
1. Faça login com qualquer usuário
2. Crie uma transação no formulário
3. Verifique no Firebase Console se ela aparece em `transactions/{userId}/`
4. A transação deve aparecer **imediatamente** no banco

### Teste 2: Sem localStorage
1. Abra DevTools (F12) → aba Application → Local Storage
2. Verifique que NÃO existe chave `financial-app-storage`
3. Crie uma transação
4. Verifique novamente - ainda não deve haver dados no localStorage
5. Todos os dados estão apenas no Firebase

### Teste 3: Sincronização entre abas
1. Abra o app em duas abas do navegador
2. Faça login com o mesmo usuário nas duas
3. Crie uma transação na primeira aba
4. A segunda aba deve mostrar a transação **automaticamente** (real-time)

### Teste 4: Recarregar página
1. Crie algumas transações
2. Recarregue a página (F5)
3. Faça login novamente
4. Todas as transações devem aparecer (carregadas do Firebase)

## 🆘 Troubleshooting

### Transações não aparecem no Firebase
1. Verifique se o `databaseURL` está correto no `firebaseConfig`
2. Abra o Console do navegador (F12) e procure por erros
3. Verifique se as regras do Firebase permitem escrita

### Erro de permissão
- Se estiver em modo de produção, ajuste as regras de segurança
- Em desenvolvimento, pode usar modo de teste

### Transações duplicadas
- Não deve mais ocorrer (localStorage removido)
- Se ocorrer, verifique se há múltiplas abas abertas
- Cada aba tem seu próprio listener real-time

## 🏗️ Arquitetura do Sistema

### Fluxo de Dados

```
┌─────────────────────────────────────────────────────────────┐
│                    FIREBASE REALTIME DATABASE               │
│  (Fonte única da verdade - Single Source of Truth)         │
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
               │ Login: Carrega dados         │ Real-time: 
               │ Create: Salva dados          │ Sincroniza
               ↓                              ↓
┌──────────────────────────────────────────────────────────────┐
│                      ZUSTAND STORE                           │
│              (Estado em memória - sessão)                    │
│  - currentUser                                               │
│  - transactions (array)                                      │
│  - accounts (array)                                          │
│  - globalFilter                                              │
└──────────────┬───────────────────────────────────────────────┘
               │
               │ React components
               │ consomem via hooks
               ↓
┌──────────────────────────────────────────────────────────────┐
│                    COMPONENTES REACT                         │
│  - Dashboard                                                 │
│  - TransactionForm                                           │
│  - TransactionsPage                                          │
│  - etc.                                                      │
└──────────────────────────────────────────────────────────────┘
```

### Características

✅ **Zero localStorage** - Nenhum dado persistido no navegador  
✅ **Real-time sync** - Múltiplos dispositivos sincronizados automaticamente  
✅ **Session-based** - Estado limpo ao recarregar (deve fazer login)  
✅ **Single source of truth** - Firebase é a única fonte de dados  
✅ **Offline-first ready** - Pode adicionar offline support do Firebase no futuro
