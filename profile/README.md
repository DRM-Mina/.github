# DRM Mina 🎮
## Game Marketplace with DRM protection in Mina

DRM Mina is a game marketplace that connects game creators and players on Mina. It uses Mina's zk-proofs as an alternative to today's anti-piracy systems, preventing authorized or unauthorized use of 3rd party applications and users' data, as well as removing the dependency of game ownership on game markets, preventing monopolies and high deductions from game developers. 

## How It Works

#### Buying game from market and registering device to play

DRM Mina app-chain contains 3 main modules that:

`Balances`: control user balance (in Mina) to buy games.

`GameToken`: allow creators to publish their games, customize them and controls the sales of games.

`DRM`: keep device information (their hash of course just for verify proofs) of users that have bought game, and their device's last session informations. 

![Buy and register game](https://github.com/DRM-Mina/.github/assets/67785258/295aff03-8463-45b2-a7b3-ae10ab874626)

First, game producers register their games in the system using `GameToken` with a submission fee (currently 0 fee in testnet). In doing so, they set their own properties such as: 
```
gamePrice
discount
timeoutInterval
number_of_devices_allowed
``` 
and only they can change them afterwards. In this way, users can purchase the game added to the system. 

Users purchase their games or can pay for their frinds and gift to them on Mina by paying the sales amount `gamePrice - discount` set by the publisher with Mina (Figure - 1, Arrow - 1). Users who have purchased the game are stored as:
```
[key: (gameId, UserPublicKey)] -> value: Bool
```
After buying the game (even *without buying* it, or **from any source** 🤯) users can download games to their computers. However, they will need to register their device in order to play. To submit their devices, they use zk-proofs with a built-in system to ensure privacy. This not only ensures privacy but also prevents them from spoofing information they don't have. (Figure - 1, Arrow - 2)
These are the unique identifiers that our system uses to distinguish devices from each other:
```
cpuId
systemSerial
systemUUID
baseboardSerial
macAddress
diskSerial (not supported yet, but soon)
```
![device identifier proof](https://github.com/DRM-Mina/.github/assets/67785258/b2c322a0-7f30-4682-9fa8-d1ab5a7b93ad)

After the device information is collected (**without taking it out of the user's computer**), the hash is created with the zk-program to save it by passing it through some calculations such as, does the information being saved meet the specified criterias? (Figure-2)

This proof allows the device to be registered in any slot of the user's choice from the number of slots specified by the game publisher. `min: 1, max: 4 slots` (Figure-1, Arrow-3 to *DRM Module*)

Now your device is set and ready to go 🚀

### Session creation and playing 

How is ownership verified when players enter the game they downloaded? DRM Mina uses its own DRM protection to solve this in the most player-friendly, flexible and secure way possible. The verification process consists of 3 main parts: 
```
App-chain
Game and its verification scripts
Local Nodejs host for generating zk-proofs and Mina txs 
```
![Create Session](https://github.com/DRM-Mina/.github/assets/67785258/86d1549d-bd02-486b-9694-3aeb37b94399)


When the user first opens the game, the game verifier takes the device's information in the background and runs some checks first, then it sends a query to the app-chain sequencer to get and check the current session information on the chain. If the game is enabled, it selects a random value that is different from the current one and sends the necessary information to the Nodejs host (which runs beside the game) and asks it to update the state and continuously checks the on-chain state for a certain period of time.

### Session Proof Generation and Submission

As public input when generating session proof it takes:
```
gameId
currentSessionKey
newSessionKey
```
In addition to these, it also receives the information that we do not want to leave the user's computer but at the same time we need to make sure that it is generated from the correct device as private input. After providing the necessary checks, it gives `gameId, newSessionKey, hash` as public output. In this way, both confidential information is prevented from leaving the computer and fraud attempts or double usage of proofs that can be made while updating the session state are prevented.

![Device Session Proof](https://github.com/DRM-Mina/.github/assets/67785258/6965b892-35b1-4641-89aa-6c36a2c2d475)

The user's wallet is not needed to send this transaction. The host can send the transaction to the sequencer from the random temporary wallet it created. The sent tx changes the state if it contains a valid proof. If the in-game validator validates it within the specified time, the player can play the game without generating a new session during the `timeoutInterval` but if not, the player is logged out from game. 
