git clone https://github.com/MarcMaverick/Toffix – Laffite – Token –
cd parufyx-token

mkdir contracts
mkdir scripts
npm init -y
PARUFYX.sol, PARUFYXGovernor.sol, deploy.js, hardhat.config.js

# Jetzt Dateien reinlegen:
# PARUFYX.sol, PARUFYXGovernor.sol -> contracts/
# deploy.js -> scripts/
# hardhat.config.js -> ins Hauptverzeichnis (parufyx-token/)

npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npm install @openzeppelin/contracts

npx hardhat compile
