# Five Stones Game Using Multithread Programming

프로젝트 링크 : https://github.com/newtron-vania/FiveStonesGame

## 자바 스윙을 이용한 멀티스레딩 기반 오목게임

오목은 우리가 어렸을 적부터 해왔던 간단하지만 재미있는 규칙을  가진 놀이다.  소켓 프로그래밍을 이용하여 호스트 간의 데이터의 전달이 가능한 것을 이용하여 다수의 호스트 간의 채팅과 오목을 할 수 있는 로비와 오목 알고리즘을 구현하였다.


### 프로젝트 개발목적
인터넷 혹은 로컬로 모여 다른 사람과 즐길 수 있는 오목 프로그램

### 프로젝트 특징
대기실을 만들어 프로그램을 실행한 사람들이 대기하고 각자 게임을 위한 방을 생성하는 멀티룸 형식 프로그램 

![캡처](https://user-images.githubusercontent.com/48934539/201453668-33c9567f-b2a4-472d-8637-df13ddd34678.PNG)

### 주요 기능
- 로그인 시 로비로 입장한다. 접속한 호스트 리스트와 게임을 하기 위해 생성된 게임 로비 리스트를 출력한다. 접속한 호스트끼리의 채팅이 가능하다.

<img src="https://user-images.githubusercontent.com/48934539/201453895-1bc873f3-c15e-4b84-8821-17635eb15fd3.png" width="600" height="400" >


- 1대1 대화가 가능한 게임 로비를 생성할 수 있다. 주인(king)과 손님(sub)가 존재할 경우 그 로비는 로비 리스트에서 제거된다.
- sub가 게임 로비에서 나갈 경우, sub자리가 비게 되어 게임 로비 리스트에 다시 추가된다. king이 게임 로비를 나갈 경우, sub 또한 게임 로비를 자동적으로 나가게 되며 그 게임 로비는 소멸한다.

<img src="https://user-images.githubusercontent.com/48934539/201454225-021c82b5-1222-4149-9372-e3ed9c561146.png" align="center" width="600" height="400" >


<img src="https://user-images.githubusercontent.com/48934539/201454306-bf9ada92-0110-443d-82d7-ba0ba10b01df.png" align="center" width="300" height="400" >



- 게임 로비의 king과 sub가 모두 존재할 경우, king은 게임을 시작할 수 있다. 시작 시 게임 룸으로 이동하며 게임은 king이 흑돌로 선공, sub가 백돌로 후공이다.

- 게임 룸은 채팅과 시스템 메시지를 보여주는 채팅 로그창과 바둑판으로 구성되어 있다. 바둑판은 29cellx 29cell 크기를 기준으로 19 x 19로 구성되어 있으며 각 선이 마주치는 점을 제외한 다른 곳에 돌을 둘 수 없다. 

<img src="https://user-images.githubusercontent.com/48934539/201454565-1a1c4032-dd2f-456e-bec1-232facd56e15.png" align="center" width="700" height="600" >



- 턴이 주어질 때마다 1개의 돌을 둘 수 있으며, 돌을 둘 때마다 오목이 오목이 완성되었는지 체크하고, 미완성되었으면 상대에게 턴을 넘긴다. 본인의 턴이 아닐 때 둘 경우 경고 메시지를 본인의 채팅 로그에 출력하고 행동을 취소한다.

<img src="https://user-images.githubusercontent.com/48934539/201454689-a14142ee-58d3-4e95-915c-281ae8ad9229.png" align="center" width="700" height="600" >

<img src="https://user-images.githubusercontent.com/48934539/201454693-aa437b38-3003-4839-bd9a-503455d66f6b.png" align="center" width="700" height="600" >


- 오목이 완성하였을 때, 승자에게는 승리 메시지가, 패자에게는 패배 메시지가 출력된다. 메시지 클릭 시 게임은 초기화되며 게임을 다시 시작한다.

<img src="https://user-images.githubusercontent.com/48934539/201454776-e9fdaaea-e542-483a-8c39-d1cea7c7b4c5.png" align="center" width="700" height="600" >


### 적용 기술

#### 소켓 프로그래밍 : 채팅과 데이터의 실시간 양방향 통신을 구현하기 위해 소켓 방식을 채용했다. 실시간 변화에 따라 호스트 간의 데이터와 메시지 송수신이 실시간으로 확인된다.
#### 브로드캐스트 : 다중 채팅을 구현할 때 트래픽 최소화를 위해 TCP/IP 기반의 다중 소켓 방식이 아닌 UDP기반의 브로드캐스팅 방식을 채용했다.
```java
/*멀티캐스트 그룹에 합류시킴*/
public ArrayList<GameServer> getCollection(int roomNumber) {
		ArrayList<GameServer> list = new ArrayList<GameServer>();
		
		for (GameServer temp : clientList)
			if(temp.getRoomNumber() == roomNumber)
				list.add(temp);
		
		return list;
		
	}

 	public void broadcasting(Protocol data) {
		System.out.println("in broadcast : " + data);
		LogFrame.print("in broadcast : " + data);
		
		for (GameServer temp : userList.getCollection())
			if(temp.getUserLocation() == ServerInterface.LOBBY) {
				try {
					temp.sendMessage(data);
				} catch (Exception e) {
					userList.subUser( temp );
					System.out.println(temp.getUserName() + "E.Lobby01");
				}
			}
	}
```

#### 오목 알고리즘 : java.awt.Point 클래스를 이용하여 입력받는 포인트 좌표를 저장한다. 돌을 둘 때마다 포인트 좌표에 추가되어 바둑돌 리스트에 추가된다.
```java
	public int addStone(int[] stoneLocation, boolean isBlack) {
		int x = stoneLocation[X] / 29;   //  '29' is size of cell.
		int y = stoneLocation[Y] / 29;
		
		pointHistory.add(new Point(x,y));
			
		if (isBlack)
			this.board[x][y] = 0;
		else this.board[x][y] = 1;
		

		return analysis(x, y);
	}
```
돌을 둘 때마다 현재 놓은 돌 위치를 기준으로 팔방으로 연속으로 존재하는 같은 색깔이 돌이 몇개가 존재하는지 확인한다. 
```java
/*오목인지 확인*/
private int analysis(int xStart, int yStart) {
		int[] point = new int[2];
		point[0] = xStart;
		point[1] = yStart;
		
		int up = 0;
		int rightUp = 0;
		int right = 0;
		int rightDown = 0;
		int down = 0;
		int leftDown = 0;
		int left = 0;
		int leftUp = 0;
		
		int[] result = new int[4];

		up        = countSameStone(point, UP);        //(0,-1)
		rightUp   = countSameStone(point, RIGHT_UP);  //(1,-1)
		right     = countSameStone(point, RIGHT);     //(1,0)
		rightDown = countSameStone(point, RIGHT_DOWN);//(1,1)
		down      = countSameStone(point, DOWN);      //(0,1)
		leftDown  = countSameStone(point, LEFT_DOWN); //(-1,1)
		left      = countSameStone(point, LEFT);      //(-1,0)
		leftUp    = countSameStone(point, LEFT_UP);   //(-1,-1)
		
		result[0] = up + down;
		result[1] = rightUp + leftDown;
		result[2] = right + left;
		result[3] = rightDown + leftUp;
		
		for (int i = 0; i < result.length; i++) {
			if(result[i] == 4) return 1;
		}
		
		return -1;
	}
```
```java
/*좌표를 기준으로 방향을 받아 같은 색의 연속으로 존재하는 돌인지 몇 개인지 확인한다. 바둑판 맨 끝에 도달하거나 연속이 아닐 시 중지하고 개수 반환*/ 
	private int countSameStone(int[] point, int[] direction) {
		int[] checkPoint = new int[2];
		
		checkPoint[0] = point[0];
		checkPoint[1] = point[1];
		
		int count = 0;

		while (true) {
			checkPoint[X] += direction[X];
			checkPoint[Y] += direction[Y];
			
			if (checkPoint[X] < 0 || checkPoint[Y] < 0)       // pointer reach edge of board
				break;
			else if (checkPoint[X] > 18 || checkPoint[Y] > 18)
				break;
					
			if(checking(point, checkPoint)) 
				count++;
			else break;
		}
		return count;
	}
```
```java
/*지정된 위치의 돌의 색깔이 같은지 확인*/
	private boolean checking(int[] point, int[] checkPoint) {
		int x = point[X];
		int y = point[Y];
		int x_check = checkPoint[X];
		int y_check = checkPoint[Y];
		
		try {
			if(board[x][y] == board[x_check][y_check])
				return true;
			else {
				return false;
			}
		} catch(ArrayIndexOutOfBoundsException e) {
			System.out.println("x : " + x_check);
			System.out.println("y : " + y_check);
			System.out.println("ArrayIndexOutOfBoundsException!!");
			return false;
		}
		
	}
```


### 개선할 기능
- 오목 무르기 추가
- 인공신경망(CNN) 기반의 오목 인공지능 추가
- 게임 로비로 복귀 시 먼저 나온 사용자 쪽 채팅 표현 안됨
- 데이터베이스에 각 사용자의 정보를 추가 기록할 필요가 있음 ex)사용자 이미지, 승패 기록, 생성 날짜 등
