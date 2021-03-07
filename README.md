# Changeling Server
the Websocket server for the Changeling game, this readme includes the overview of the stack used for the websocket server and the protocol used in the connection itself.

## Server Structure

Changeling's server uses Python 3.7 with the `websockets` package for easy prototyping, this package is more than enough to handle the data traffic the game will produce.

### Protocol

Changeling's protocol is a JSON protocol, each message between the client and the server carries the minimum amount of information, the server is authorative over the client to the point that the client acts similar to a dummy terminal.

#### Client Requests 

All messages from client to server are **requests**, following requests can be sent:

* Request to create a new game.
* Request to join an existing game.
* Request to start the game from its lobby.
* Request to restart the current game after it finished.
* Request to finish one's own turn.
* Request to vote on a camper.
* Request to turn a camper to a changeling.

##### Request to create a new game

| Request Name | `req_host_game`|
|--------------|----------------|
|From| User in login screen.|

###### Preconditions

* User must not have logged in.
* User must not be on a room other than their own.

###### Postconditions

* Add user to the room as the admin.
* Create a room with a new room ID.

###### Format

```json
{
  "name": "username",
  "portrait": "portrait_id"
}
```

##### Request to join an existing game

| Request Name | `req_join_game`|
|--------------|----------------|
|From| User in login screen.|

###### Preconditions

* User must not have logged in.
* User must not be on a room other than their own.
* The room ID must be valid.
* The room must have under five members.

###### Postconditions

* Send user a `resp_join_room` response.
* Append the user to the new room.
* Upon user joining the room sync up the game state by sending them information on other users.

###### Format

```json
{
  "name": "username",
  "portrait": "portrait_id",
  "roomID": "roomID"
}
```

##### Request to start the game from lobby.

| Request Name | `req_join_game`|
|--------------|----------------|
|From| Room admin.|

###### Preconditions

* User must be the admin of the room they are in.

###### Postconditions

* Assign each user a role.
* Sync up the game states.

###### Format

No format, empty event.

#### Server Updates

Server provides the following updates to sync up the game state between the clients:

* Update the player list.
* Update the room id
* Update to player state.
* Update the turn number.
* Update the game state.

Therefore, there is no single request, instead, each request has its own format, albeit they all share one common field.

As the Server has the authoritive role in the connection, client's state is in the state is "shallow", that is, the server to client messages are incremental updates sent to each client. The server does not have to hold the game state from a single client's POV, as:

1. Before the game starts, in the lobby, barring the initial join, the game state for each player is equal.
2. Within the game, the game state is equal between all players, only difference being changelings can see other changelings.

Therefore there are two special cases in the application that necessitates.

1. When a player joins a room for the first time, server sends the already existing player list by sending multiple player-list-updates to that player.
2. When a player becomes a changeling, server sends multiple

##### Update Room ID (Acknowledge Game Host)

| Response Name | `resp_ack_host` |
| ------------- | --------------- |
| To            | Room admin.     |

###### Preconditions

* Must occur from a host game request.

###### Postconditions

* User is sent their room ID, as well as admin setup.

###### Format

```json
{
    "room_id": "ABCDE",
    "is_you": true,
    "user_id": "user_id",
    "name": "user_name",
    "portraitName": "portrait_id",
    "playerRole": "playerRole"
    "admin": true
}
```



##### Acknowledge Game Join

| Response Name | `resp_ack_join`                       |
| ------------- | ------------------------------------- |
| To            | Any player that just joined the room. |

###### Preconditions

* Must occur from a  join game request.

###### Postconditions

* User is sent back the ID of the room they wanted to join in the first place.

###### Format

```json
{
    "roomID": "ID of the room user wanted to join."
}
```



##### Update Player List  (Add)

| Response Name | `resp_join_player`                         |
| ------------- | ------------------------------------------ |
| To            | Members of a room when a new player joins. |

###### Preconditions

* Must occur from a host or  join game request.

###### Postconditions

* User is sent info about other players via same event.
* Other players mut be informed.

###### Format

```json
{
    "is_you": true,
    "user_id": "user_id",
    "name": "user_name",
    "portraitName": "portrait_id",
    "playerRole": "playerRole"
    "admin": true
}
```



