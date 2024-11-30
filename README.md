# Fractal-CAT20-Pumpfun-Backend
The backend for pump.fun on the Fractal Bitcoin Network, supporting the Cat Protocol, is built with Node.js and sCrypt. It functions as a launchpad for CAT20 tokens, enabling their minting, trading, and transfer operations.

## Contact Info
If you need more technical support and development inquires, you can contact below.

Telegram: @dwlee918

X: @derricklee918

Discord: @solGuru

**Installation:**

```bash
# Clone the repository 
git clone https://github.com/solguru310/Fractal-CAT20-Pumpfun-Backend.git

# Install dependencies 
cd Fractal-CAT20-Pumpfun-Backend
npm install 
```

### Core Functionality 

**Token Minting API Fetching:**
```javascript
const { mintToken } = require('./services/tokenService');

(async () => {
  try {
    const result = await mintToken('tokenName', 'tokenSymbol', 1000);
    console.log('Token minted:', result);
  } catch (error) {
    console.error('Error minting token:', error);
  }
})();
```

This function utilizes a service to mint CAT20 tokens, specifying the token details.

**Trading Tokens:**
```javascript
const { tradeToken } = require('./services/tradeService');

(async () => {
  try {
    const result = await tradeToken('tokenId', 'toAddress', 100, 'traderPrivateKey');
    console.log('Token traded:', result);
  } catch (error) {
    console.error('Error trading token:', error);
  }
})();
```

This snippet is a demonstration of trading tokens using a specified trading service.

**Retrieving Token Information:**
```javascript
const fetch = require('node-fetch');

async function getTokenInfo(tokenId) {
  const response = await fetch(`https://api-pump.fun/info?token_id=${tokenId}`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.API_KEY,
    },
  });
  const data = await response.json();
  return data;
}

(async () => {
  try {
    const info = await getTokenInfo('tokenId');
    console.log('Token Info:', info);
  } catch (error) {
    console.error('Error fetching token info:', error);
  }
})();
```

This is guide on structuring the Node.js backend for launching and trading CAT20 tokens on the Fractal Bitcoin Network. This will include examples for the model, controller, and router components using JavaScript and Express.js. 

## Backend Infrastructure Codebase Parts

### 1. Model

The model is responsible for defining the CAT20 token structure and any methods associated with it. Below is an example using a simple class-based approach.

**models/CAT20Token.js:**
```javascript
class CAT20Token {
  constructor(name, symbol, totalSupply, owner) {
    this.name = name;
    this.symbol = symbol;
    this.totalSupply = totalSupply;
    this.owner = owner; // Address of the token owner/creator
    this.balances = { [owner]: totalSupply }; // Track token balances
  }

  transfer(from, to, amount) {
    if (this.balances[from] >= amount) {
      this.balances[from] -= amount;
      if (this.balances[to]) {
        this.balances[to] += amount;
      } else {
        this.balances[to] = amount;
      }
      return true;
    }
    throw new Error('Insufficient balance');
  }
}

module.exports = CAT20Token;
```

### 2. Controller

Controllers handle the business logic and interact with models to process requests. Below is an example of a controller handling token minting and trading.

**controllers/CAT20TokenController.js:**
```javascript
const CAT20Token = require('../models/CAT20Token');

// In-memory storage of tokens for demonstration
const tokens = {};

exports.createToken = (req, res) => {
  const { name, symbol, totalSupply, owner } = req.body;
  if (tokens[symbol]) {
    return res.status(400).json({ message: 'Token symbol already exists' });
  }
  const token = new CAT20Token(name, symbol, totalSupply, owner);
  tokens[symbol] = token;
  res.status(201).json({ message: 'Token created', token });
};

exports.tradeToken = (req, res) => {
  const { symbol, from, to, amount } = req.body;
  const token = tokens[symbol];
  if (!token) {
    return res.status(404).json({ message: 'Token not found' });
  }
  try {
    token.transfer(from, to, amount);
    res.status(200).json({ message: 'Tokens transferred successfully', token });
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};
```

### 3. Router

The router handles the HTTP requests and is used by the Express app to call the appropriate controller methods.

**routes/CAT20TokenRoutes.js:**
```javascript
const express = require('express');
const router = express.Router();
const CAT20TokenController = require('../controllers/CAT20TokenController');

router.post('/create', CAT20TokenController.createToken);
router.post('/trade', CAT20TokenController.tradeToken);

module.exports = router;
```

### 4. Server Setup

Finally, ensure that these components are tied together within your Express application. 

**server.js:**
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const CAT20TokenRoutes = require('./routes/CAT20TokenRoutes');

const app = express();
app.use(bodyParser.json());

app.use('/api/tokens', CAT20TokenRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```
