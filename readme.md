# WellAI
![Logo](https://github.com/GaLenN3228/wellai_mobile/blob/master/assets/logo.png?raw=true)

## About app

WellAI is an app for patients and doctors from Bent Tree Family. This is one of the best clinics in Frisco and North Dallas. The app contains a lot of features which profile in the "Features" section. We split doctor and patient roles into two apps because all functions are specified for role, it will be hard and complicated to hold all roles in one app. Because of that, app for doctor contains nurse and doctor role. At the same time doctor can be a patient with the same email. 

## 🛠 App languages
Mobile app: Dart and flutter<br />
Backend: Go<br />
Web app: Vue.js

## Features

- Chat with nurse or doctor  [example]()
- Virtual assistant to help with diagnoses   [example]()
- WebRTS calling to nurse or doctor  [example]()
- Authorization with google or apple [example]()
- Attach to doctor or nurse in calendar [example]()
- Profile editing [expample]()

## Chat with nurse or doctor

Chat with nurse or doctor implementation 

For chat system we use Web socket for connection and handling messages. For WebSocket connection we use [socket_io_client](https://pub.dev/packages/socket_io_client). The connection example provided here:

```dart
    _socket = socket_io.io(
        baseUrl,
        <String, dynamic>{
          'transports': ['websocket'],
          'reconnection': true,
          'timeout': 20000,
          'forceNew': true,
          'auth': {'token': 'Bearer ${_tokensRepository.accessToken}'},
        },
      );
      _socket.onConnectError((data) {
        log('connect error', name: _tag);
      });
      _socket.on('error', (data) async {
        log(data.toString(), name: _tag);
      });
      _socket.onDisconnect((data) {
        log('disconnect', name: _tag);
      });
      _socket.onReconnect((data) {
        log('reconnect', name: _tag);
        _initWebRTC();
      });
      _socket.onConnect((data) {
        networkInfo.updateConnection(true);
        log('onConnect', name: _tag);
        _initWebRTC();
      });
```

Main event from socket handler: 

```dart
     void _initWebRTC() {
    _onEvent(WsCancelResponse.name);
    _onEvent(WsCallResponse.name);
    _onEvent(WsMediaOfferResponse.name);
    _onEvent(WsMediaAnswerResponse.name);
    _onEvent(WsApproveResponse.name);

    _onEvent(WsChatSendedMessageResponse.name);
    _onEvent(WsJoinChatResponse.name);
    _onEvent(WSLeaveChatResponse.name);
    _onEvent(WsRemotePeerIceCandidateResponse.name, checkFunction: (data) {
      return data['candidate'] != null;
    });
    _onEvent(WsChatMessageResponse.name);
    for (var event in WSScreenUpdateResponse.subValues) {
      _onEvent(event);
      emit(event, SimpleEncodable({}));
    }
  }
```

and _onEvent fucntion: 

```dart
    _onEvent(
        String event, {
        bool Function(dynamic)? checkFunction,
        VoidCallback? customCallBack,
    }) {
        _socket.off(event);
        _socket.on(event, (data) async {
        log('SocketEvent: ' + event);
         log('Event data: ${jsonEncode(data)}');
        if (checkFunction?.call(data) ?? true) {
            _eventStreamController.sink.add(WsResponse.fromMap(event, data));
            customCallBack?.call();
        }
    });
  }
```

Because chat bloc have complicated structer, we provide only inmplementation, in this example you can find connection to chat, and listener method, whitch listen events from previous part. All complited example can be found in "Features" section: 

```dart
  NurseChatBloc(
    this._chatWrapper,
    this._viewModel,
    this._repository,
    this._userStore,
    this._networkInfo,
  ) : super(LoadingNurseChatState()) {
    on<SendMessageEvent>(_onSendMessageEvent);
    on<GetMessageEvent>(_onGetMessageEvent);
    on<InitialNurseChatEvent>(_onInitialNurseChatEvent);
    on<ReconnectNurseChatEvent>(_onReconnectNurseChatEvent);
    _messagesSub = _chatWrapper.messagesStream.where((event) {
      final ev = event as WsChatMessageResponse;
      return ev.chatId == _chatId;
    }).listen((event) {
      add(GetMessageEvent(event as WsChatMessageResponse));
    });
    _networkConnectionSub = _networkInfo.reconnectionStream.listen((event) {
      if (event == true) {
        add(ReconnectNurseChatEvent());
      }
    });
  }
```