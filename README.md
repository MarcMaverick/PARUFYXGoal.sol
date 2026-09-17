# parufyx-token
PARUFYX – ERC20-Governance-Token (ERC20Votes) mit Timelock-Governor auf Basis von OpenZeppelin v5
git clone https://github.com/DEIN-USERNAME/parufyx-token.git
cd parufyx-token
mkdir contracts
mkdir scripts
parufyx-token/
├── hardhat.config.js          ← hier, im Root
├── package.json
├── contracts/
│   ├── PARUFYX.sol
│   └── PARUFYXGovernor.sol
└── scripts/
    └── deploy.js
    npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox @openzeppelin/hardhat-upgrades
npm install @openzeppelin/contracts
npx hardhat compile
git add .
git commit -m "Initial commit: PARUFYX token + governance contracts"
git push origin main
