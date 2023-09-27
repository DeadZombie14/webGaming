# webGaming

This is a little project I will be working on that consists on a self hosted web platform that allows you and your friends to play any Windows app together using web streaming.

The main core technologies are WebRTC through the library PeerJS (https://github.com/peers/peerjs), web gamepad API implementation from MDN and finally ViGemBus + NetJoy (https://github.com/Qcent/NetJoy/), which allows me to create virtual pads on demand in the windows host computer.

This is all inspired by the sudden removal of remote play together in steam, which left me and my friends with no way to play these games online, and parsec aint an option.

We'll see how it goes! 

## TODO list
- Find a suitable web server that can ease the setup process (start both the client web server and peerjsserver at the same time)
- Decide on a web framework (vanilla js aint that good)
- Modify NetJoy receiver app so it stablishes web socket connections to main WebRTC host
- Apply CSS on whole site

