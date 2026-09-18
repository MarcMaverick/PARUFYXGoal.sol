https://github.com/MarcMaverick/Toffix-Laffite-Token-.git
cd parufyx-token

mkdir contracts
mkdir scripts

# Jetzt Dateien reinlegen:
# PARUFYX.sol, PARUFYXGovernor.sol -> contracts/
# deploy.js -> scripts/
# hardhat.config.js -> ins Hauptverzeichnis (parufyx-token/)

npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npm install @openzeppelin/contracts

npx hardhat compile
