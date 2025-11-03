# KipuBank v2 — Contrato Inteligente Multiactivo y Seguro

## Descripción General

**KipuBank v2** es una evolución del contrato original **KipuBank**, diseñado para acercarse a un entorno de producción real.  
La nueva versión introduce mejoras sustanciales en **seguridad**, **extensibilidad**, **control administrativo** y **contabilidad multi-activo**, integrando prácticas avanzadas de desarrollo en Solidity y herramientas estándar de la industria (OpenZeppelin y Chainlink).

El contrato mantiene la esencia del banco descentralizado:
- Los usuarios depositan y retiran activos (ETH o ERC-20).
- Los fondos se almacenan en bóvedas personales.
- Se aplican límites individuales y globales para proteger la integridad del sistema.

---

## Mejoras Principales y Motivación

### 1. Control de Acceso (OpenZeppelin AccessControl)
**Motivo:** En la versión original, cualquier usuario podía potencialmente cambiar parámetros críticos si se agregaban nuevas funciones en el futuro.  
**Solución:**  
- Se introdujeron roles con permisos diferenciados:
  - `DEFAULT_ADMIN_ROLE`: configuración y mantenimiento.
  - `MANAGER_ROLE`: operaciones como registrar oráculos o ajustar parámetros.  
- Esto mejora la seguridad operativa y facilita delegar funciones sin comprometer el control global.

---

### 2. Soporte Multi-Token (ETH + ERC-20)
**Motivo:** El contrato original solo manejaba ETH. En un entorno real, se necesita aceptar distintos activos (USDC, DAI, WETH…).  
**Solución:**  
- Se agregó soporte para **depósitos y retiros de tokens ERC-20**.
- Se usa `address(0)` como identificador del token nativo (ETH).
- Se implementó una contabilidad interna **multi-token**, donde cada usuario mantiene balances independientes por token.

---

### 3. Contabilidad Interna en USD (con Chainlink)
**Motivo:** En la primera versión, el límite (`bankCap`) estaba expresado en ETH. Esto no reflejaba su valor real en dólares, que varía con el tiempo.  
**Solución:**  
- Se integraron **Chainlink Data Feeds** (por ejemplo `ETH/USD`).
- Todos los límites (`bankCap` y `perTxWithdrawLimit`) se expresan en **USD**.
- Cada depósito convierte el monto depositado a USD usando el oráculo correspondiente.  
Esto permite mantener un **control económico real** del sistema, independientemente de la volatilidad del mercado.

---

### 4. Manejo de Decimales y Precisión
**Motivo:** Los tokens ERC-20 pueden tener diferentes decimales (6, 8, 18...).  
**Solución:**  
- Se implementó una función interna que normaliza todos los montos a **6 decimales estándar (como USDC)** para la contabilidad interna.
- Evita inconsistencias en cálculos y límites al comparar valores entre tokens.

---

### 5. Seguridad y Buenas Prácticas
**Motivo:** Preparar el contrato para escenarios reales de interacción y evitar vulnerabilidades comunes.  
**Solución:**  
- Implementación estricta del patrón **Checks → Effects → Interactions**.
- Uso de **errores personalizados** en lugar de `require` con strings (optimiza gas y legibilidad).
- Uso de **transferencias nativas seguras** con `call`.
- Variables críticas (`bankCapUSD`, `perTxWithdrawLimitUSD`) declaradas como `immutable`.
- Integración de **OpenZeppelin ReentrancyGuard** para proteger contra ataques de reentrancia.

---

### 6. Observabilidad y Auditoría
**Motivo:** Mejorar trazabilidad de transacciones para monitoreo y debugging.  
**Solución:**  
- Nuevos eventos:  
  - `Deposit(address user, address token, uint256 amount)`
  - `Withdrawal(address user, address token, uint256 amount)`
  - `OracleUpdated(address token, address feed)`
- Estos eventos permiten auditar depósitos, retiros y actualizaciones de oráculos desde interfaces como Etherscan o TheGraph.

---

## Instrucciones de Despliegue

### 🔧 Requisitos previos
- **Remix IDE** o **Hardhat**.
- **Solidity 0.8.19** o superior.
- Si se usa una testnet (p. ej., **Sepolia**), asegúrate de tener:
  - ETH de prueba.
  - Dirección del **Chainlink ETH/USD Price Feed**:
    - Sepolia: `0x694AA1769357215DE4FAC081bf1f309aDC325306`

---

### Despliegue en Remix

1. Abre [Remix IDE](https://remix.ethereum.org).
2. Crea un archivo `KipuBankV2.sol` y pega el código del contrato.
3. Compila con **Solidity 0.8.19** o superior.
4. En la pestaña **Deploy & Run**:
   - Environment: `Injected Provider - MetaMask` (para testnet) o `Remix VM (London)` (para pruebas locales).
   - En los parámetros del constructor, ingresa:
     ```
     _bankCapUSD = 100000000000     // 1,000,000 USD (8 decimales)
     _perTxWithdrawLimitUSD = 200000000  // 2,000 USD
     _ethUsdFeed = 0x694AA1769357215DE4FAC081bf1f309aDC325306
     ```
   - Haz clic en **Deploy**.

---

### 🧪 Interacción Básica

#### Depositar ETH
1. En Remix, selecciona el contrato desplegado.
2. En el campo **Value**, escribe la cantidad de ETH (ej. `1 ether`).
3. Llama a:
