# Unity
## 1. 预制体、预制体变体
## 2. Vector3
```c#
//向量，坐标，旋转，缩放
Vector3 v= new Vector3(1,1,1)
Vector3.Angle（v，v2）//夹角
Vector3.Distance（v，v2）//距离
Vector3.Dot（v，v2）//点乘
Vector3.Cross（v，v2）//叉乘
Vector3.Cross（Vector3.zero，Vector3.one, 0.8f）//插值
```
## 3. 旋转：欧拉角，四元数
```c#
Vector3 rotate = new Vector3(0,30,0)
Quaternion quaternion=Quaternion.identity;
quaternion = Quaternion.Euler(rotate);//欧拉角转四元数
rotate= quaternion.eulaerAngles;//四元数转欧拉角
quaternion = Quaternion.LookRotation(new Vectors(0,0,0));//看向一个物体

//四元数，旋转
Quaternion.Euler(0,0,0)//欧拉角
Quaternion.LookRotation(Vector3.forward, Vector3.up)//朝向
Quaternion.Slerp(Quaternion.identity, Quaternion.Euler(0,0,0), 0.5f)//插值
```
## 4. 调试方法
```c#
Debug.Log();//调试
Debug.LogWarning();//警告
Debug.LogError();//错误
Debug.DrawLine(Vector3.left,Vector3.one,Color.blue)//画线
Debug.DrawRay(Vector3.zero,Vector3.up,Color.red)//射线
```

## 5. 脚本中的物体
```c#
GameObject go=this.gameObject;//当前脚本中挂载的游戏物体
Debug.Log(gameObject.name) //名称
gameObject.tag //标签
gameObject.layer //涂层
gameObjec.activelnHierarchy; //当前真正的激活状态
gameObject.activeSelf; //当前自身激活状态
Transform trans=this.transform;
Transform.position; //可以直接使用
BoxCollider bc = GetComponent<BoxCollider>(); //获取其他组件
//获取当前物体的子物体身上的某个组件
GetComponentlnChildren<CapsuleCollider>(bc);
//获取当前物体的父物体身上的某个组件
GetComponentlnParent<BoxCollider>();
Cube.AddComponent<AudioSource>();//添加一个组件
GameObject test = GameObject.Find("Test"); //通过名称获取物体
test = GameObject.FindWithTag("Enemy"); //通过标签获取物体
test.SetActive(false); //设置属性
GameObject go = Instantiate(Prefab, Vector3.zero, Quaternion.identity); //通过预制体实例化
Destroy(go); //销毁
```

## 6. 时间类
```c#
Time.time //当前时间
Time.timeScale // 时间缩放值
Time.fixedDeltaTime // 固定时间间隔
Time.deltaTime // 两帧时间间隔
```

## 7. Application:路径
```c#
Application.datapath+"文档.txt"; //游戏数据文件夹路径（只读，加密压缩）
Application.persistentDataPath; //持久化文件夹路径
Application.streaminAssetsPath; //文件夹路径（只读，配置文件）
Application.temporaryCathePath; //临时文件夹
Application.runlnBackground; //控制台是否在后台运行
Application.OpenURL("百度一下，你就知道"); //打开url
Application.Quit(); //退出游戏
```

## 8. 场景管理
```c#
SceneManager.LoadScene("MyScene");
Scene scene = SceneManager.GetActiveScene();
Scene.name; //场景名称
scene.isLoaded; //场景是否已经加载
Scene.path; //场景路径
Scene.buildindex; //场景索引
GameObject[] gos = scene.GetRootGameObjects();
Gos.Length;

Scene newScene = SceneManager.CreateScene("newScene"); //常见新场景
SceneManager.sceneCount; //已加载场景个数
SceneManager.UnloadSceneAsync(newScene); //卸载场景
SceneManager.LoadScene("MyScene", LoadSceneMode.Additive); //加载场景
```

## 9.异步加载
```c#
AsyncOperation operation;
Void Start(){
    StartCoroutine(loadScene());
}

//协程方法用来异步加载场景
Enumerator loadScene(){
    operation = SceneManager.LoadSceneAsync(1);
    operation.allowSceneActivation = false;
    yield return operation;
}

void Update(){
//输出加载进度0-0.9
    Debug.Log(operation.progress);
    timer += Time.deltaTime;
    if(timer > 5){
}
```

## 10.位置、缩放、旋转
```c#
Void Start(){
	//位置：
	transform.position
	transform.localPosition;

	//旋转：
	transform.rotaion;
	transform.localRotaion;
	transform.eulerAngles
	transform.localEulerAnles;

    //获取缩放
	transform.localScale

	//向量
	transform.forward;
	transform.right;
	transform.up;

    //父子关系
	transform.parent.gameObject; //获取父物体
	transform.childCount; //子物体个数
	transform.DetachChildren(); //解除与子物体的父子关系
	Transform trans = transform.Find("Child"); //获取子物体
	trans = transform.GetChild(0); 
	bool res = trans.IsChildOf(transform); //判断一个物体是不是另外一个物体的子物体
	Debug.Log(res);
	trans.Setparent(transform); //设置为父物体
}

Void Update(){
	//时时刻刻看向000点
	transform.LookAt(Vector3.zero);
	//旋转
	transform.Rotate(Vector3.up, 1);
	//绕某个物体旋转
	transform.RotateAround(Vector3.zero, Vector.up, 5);
	//移动
	transform.Translae(Vector3.forward * 0.1f);
}
```
## 11. 电脑游戏-键盘鼠标操作
```c#
void Update(){
	//鼠标的点击
	//按下鼠标 0左键 1右键 2滚轮
	if(input.GetMouseButtonDown(0)){
		Debug.Log("按下了鼠标左键");
	}
	//持续按下鼠标
	if(input.GetMouseButton(0)){
		Debug.Log("持续按下了鼠标左键");
	}

    //抬起鼠标
	if(input.GetMouseButtonUp(0)){
		Debug.Log("抬起了鼠标左键");
	}

	//按下键盘按键
	if(input.GetKeyDown(KeyCode.A)){
		Debug.Log("按下了A");
	}

    //持续按下按键
	if(input.GetKey(KeyCode.A)){
		Debug.Log("持续按下A");
	}
	//抬起键盘按键
	if(input.GetkeyUp("a")){
		Debug.Log("松开了A");
	}
}
```

## 12. 虚拟轴
```c#
Void Update()
	//获取水平轴
	Float horizontal = input.GetAxis("Horizontal");
	Float vertical = input.GetAxis("Vertical");
	Debug.Log(horizontal + " " + vertical);

	//虚拟按键
	if(input.GetButtonDown("Jump"){
		Debug.Log("空格");
	}
	if(input.GetButton("Jump"){
		Debug.Log("空格");
	}

	if(input.GetButtonUp("Jump"){
		Debug.Log("空格");
	}
}
```
## 13. 触摸操作
```c#
void Start(){
	input.multiTouchEnabled = true; // 开启多点触摸
}

void Update(){
	//判断单点触摸
	if(input.touchCount == 1){
		Touch touch = input.touches[0]; //触摸对象
		Debug.Log(touch.position); //触摸位置
		switch(touch.phase){
			case TouchPhase.Began:
				break;
			case TouchPhase.Moved:
				break;

			case TouchPhase.Stationary:
				break;
			case TouchPhase.Ended:
				break;
			case TouchPhase.Canceled:
				break;
		}
	}

	//判断多点触摸
	if(input.touchCount == 2){
		Touch touch = input.touches[0];
		Touch touch = input.touches[1];
	}
}
```

## 14. 灯光
    - 默认定向光、聚光、电光源、区域（仅烘培）

## 15. 摄像机，通过时合集深度（融合相机场景）设置相机优先级别
	- 透视摄像机
	- 正交摄像机
	- 针对涂层剔除遮罩
	- 视野

## 16. 声音-仅一个相机需要声音组件
	- 声乐、音效
	- 3D声音-有衰减过程
```c#
//AudioClip
Public AudioClip music; //音乐
Public AudioClip se; //音效

Private AudioSouce player; //播放器组件

void Start(){
	player = GetComponent<AudioSource>();
	player.clip = music; //设定播放的音频片段
	player.loop = true; //循环
	player.volume = o.5f;
player.Play();	
}

Void Update(){
	if(input.GetKeyDown(KeyCode.Space)){
		if(player.isPlaying){
			player.Pause();
		}else{
			player.Play();
		}
	}

	//鼠标左键播放声音（音效）
	if(input.GetMouseButtonDown(0)){
		player.PlayOneShot(se);
	}
}

## 17. 播放视频
	-创建渲染器纹理->创建平面->添加视频组件->视屏剪辑（拖拽文件）
```c#
using UnityEngine.Video;

Private VideoPlay player;
void Start(){
	player = GetComponent<VideoPlayr>();
}

void Update(){
	if (input.GetMouseButtonDown(0)){
	}
}
```

## 18. 角色控制器
	- 平面、胶囊、角色控制器组件、脚本
```c#
Private CharacterController player;
void Start(){
	player = GetComponent<CharacterController>();	
}

void Update(){
	float horizontal = input.GetAxis("Horizontal"); //水平轴
	float vertical = input.GetAxis("Vertical"); //垂直轴
	Vector3 dir = new Vector3(horizontal, 0, vertical); //创建一个方向向量
//Debug.DrayRay(transform.position, dir, Color.red);
	player.SimpleMove(dir * 2); //朝向该方向移动
}
```

## 19. 物体重力->Rigidbody组件

## 20. 碰撞的产生与监听：需要碰撞体组件，至少一个是刚体
```c#
Public GameObject Prefab;
private void OnCollisionEnter(Collision collision){
	instantiate(Prefab, transform.position, Quaternion.identity);
	Destroy(gameObject); //销毁自身
	Debug.Log(collision.gameObject.name); //获取碰撞到的物体
}

//持续碰撞中
private void OnCollisionStay(Collision collision){}
//结束碰撞
private void OnCollisionExit(Collision collision){}
```

## 21. 触发器：地面、墙、胶囊玩家、出发机关
```c#
private void OnTrggerEnter(Collider other){
	GameObject door = GameObject.Find("Door");
	if(door != null){
		door.SetActive(false);
	}
}
private void OnTrggerStay(Collider other){}
private void OnTrggerExit(Collider other){}
```
## 22. 旋转门、铰链、弹簧、特殊的物理关节

## 23. 斜面物理材质：摩擦力、弹力

## 24. 射线、红外线检测:平面、球体
```c#
void Start(){
	//Ray ray = new Ray(Vector3.zero,Vector3.up); //方式1
}

void Update(){
	//方式2
	Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
	RaycastHit hit; // 声明一个信息类
	bool res = Physics.Raycast(ray, out hit); //碰撞检测
	//如果碰撞到的情况下，hit就有内容了
	if(res == true){
		Debug.Log(hit.point);
		transform.position = hit.point;
	}

	//多检测
	//RaycastHit[] hits = Physics.RaycastAll(ray, 100, 1<<10);
}
```

## 25. 粒子火焰效果

## 26.  彩色线和拖尾
```c#
void Start(){
	LineRenderer lineRenderer = GetComponent<LineRenderer>();
	lineRenderer.positionCount = 3;
	lineRenerer.SetPosition(0,Vector3.zero);
	lineRenerer.SetPosition(0,Vector3.one);
	lineRenerer.SetPosition(0,Vector3.down);
}
```

## 27. 动画：Animation
```c#
Void Update(){
	if(Input.GetMouseButtonDown(0)){
		GetComponent<Animation>().Play("right");
	}
}
```

## 28. 动画：Animator
```c#
private Animator animator;
void Start(){
	animator = GetComponent<Animator>();
}
void Update(){
	if(Input.GetMouseButtonDown(0)){
animator.Play("right");
	}
}
```

## 29. 切换动画

    取消"有退出时间" 立即切换
```c#
void Update(){
	if(Input.GetKeyDown(keyCode.F)){
		GetComponent<Animator>().SetTrigger("pickup");
	}	
}
```

## 30. 角色控制：设置动画多渡时间，避免动画切换生硬
```c#
private Animator animator;
void Start(){
	animator = GetComponent<Animator>();	
}

void Start(){
	float horizontal = Input.GetAxis("Horizontal"); //水平轴
	flot vertical = Input.GetAxis("Vertical"); //垂直轴
	Vector3 dir = new Vector3(horizontal,0,vertical); //向量
	if(dir != Vector3.zero){ //当用户按下了方向键
		transform.rotation = Quaternion.LookRotation(dir); //面向向量
		animator.SetBool("IsRun", true); //播放跑步动画
		transform.Translate(Vector3.forward * 2 * time.detalTime); //朝向前方移动
	}else{
		animator.SetBool("IsRun", false); //播放站立动画
	}
}
```

## 31. 动画剪辑：长度、烘焙等

## 32. 动画中曲线(动画播放过程中动态值)和事件(作用的具体的动画帧上)的应用

## 33. 混合动画：多个动画根据float值过滤，走和跑动画

## 34. 动画分层：不同的层可以控制物体的不同部位

## 35. 动画捕捉：软件地址-千面视频动捕 - AI动作捕捉生成3D动画平台

## 36. 反向动力学
```c#
Public Transform target;
//IK写这个方法内
private void OnAnimatorIK(int LayerIndex){
	//设置头部IK
	animator.SetLookAtWeight(1);
	animator.SetlookAtpositon(target.position);
	animator.SetIKPositionWeight(AvatarIKGoal.RightHand,1); //设置右手IK权重
	animator.SetIKRotationWeight(AvatarIKGoal.RightHand,1); //旋转权重
	//设置右手IK
	animator.SetIKPosition(AvatarIKGoal.RightHand, target.position);
	animator.SetIKRotation(AvatarIKGoal.RightHand, target.position);
}
```

## 37. 导航系统：静态导航、窗口-AI-导航-烘焙生成可到达区域

## 38. 导航人物模型：添加人物导航代理组件（代理类型可以有多个不同类型的人）- Nav Mesh Agent
```c#
private NavMeshAgent agent;

void Start(){
    agent = GetComponet<NavmeshAgent>();
}

void Update(){
    if(Input.GetMouseButtonDown(0)){//如果按下鼠标
        //获取点击位置
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        RaycasHit hit;
        if(Physics.Raycast(ray.out hit)){
            Vector3 point = hit.point; //点击位置
            agent.SetDestination(point); //设置该位置为导航目标点
        }
    }
}
```
## 39. 动态障碍物
   - 障碍物组件（Nav Mesh Obstacle）,切割会重新烘焙
   - 烘焙：网格连接设置高度和距离，可以进行跳跃
   - Off Mesh Link（网格连接组件）两个物体之间的穿梭
## 40. 导航区域
   - 设置导航路线成本，对象绑定不同区域，区域遮罩过滤区域
## 41. UI-画布
   - Cavas渲染模式-覆盖（ui图册在前）｜摄像机（ui和物体顺序可调）｜世界（ui3D效果）
## 42. 新旧文本框设置
## 43. 按钮：
   - 导航（按键、手柄控制），事件
## 44. 输入框参数
## 45. 下拉框：
   - 每项图片添加
## 46. 滚动条、滑动框
## 47. 使用面板锚点固定对象位置
## 48. 面板内的对象设置布局组件（水平、垂直、网格）：间距设置