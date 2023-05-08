# SocketInterface

This is a simple socket interface extends from gorilla.


## Usage

#### Initial
```golang
type MsgState map[string]any
// 連線資訊
type ConnType struct {
}

// 要傳送socket 的訊息
type MessageType struct {
}
// 聊天室 的狀態
type RoomStatus struct {
	LastFinishedYearMonth string
}

// 實例化 socket
var socket = abstract.Instance[ConnType, MessageType](5) // thread(go runtine) counts

func init()  {
	// This function is a callback that allows you to store the information into your customized method.
	// Just like the redis.
	socket.EnterRoomCallBack = func(v ConnType, roomKey string) {
		(*Redis).EnterShiftRoom(v.BanchId, v.Value)
		sendMsgHandler(v.BanchId, v.User, v.Company, map[string]any{
			"newEntering": true,
		})
	};


	// This function is a callback that allows you to handle the send customized method.
	socket.SendMessageCallBack = func(
		v MessageType,
		roomKey string,
		sendMsg func(ConnId string, Msg any),
	) {
		userAll := (Redis.GetShiftRoomUser(v.BanchId)) // 獲取 該聊天室 成員
		// fmt.Print("users => ", len(*(userAll)))
		// fmt.Print("roomId => ", v.BanchId)
		for _, user := range *userAll {

			// 這是只有 自己是發起人才要傳送 錯誤訊息
			if user.UserId != v.LauchPerson.UserId {
				v.State["errorMsg"] = ""
			}

			// 根據 權限 獲取 前端 操作 狀態
			getCheckState := CheckState(v.Status, user.Permission, user.BanchId, v.BanchId)
			v.State["disabledTable"] = getCheckState["disabledTable"]
			v.State["submitAble"] = getCheckState["submitAble"]

			sendMsg(strconv.FormatInt(user.UserId, 10), v)
		}
	};
}
```



## Method
```golang
	// enterRoom
	socket.EnterRoom(
		strconv.FormatInt(connProps.User.UserId, 10),
		conn,
		connProps,
		"any",
	)

	// leave room
	socket.LeaveRoom(connId)
	
	// send message
	// this will use your cusomized sendMessageCallback methods.
	socket.SendMessage(message, roomKey)
```