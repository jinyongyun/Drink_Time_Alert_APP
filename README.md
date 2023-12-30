# 💦 물 마시기 알람 앱 만들기

Local Notification을 이용할 것이다. (다른 말로는 푸쉬 알림!) 

Local Notification은 사용자의 관심을 끄는데 사용한다.

시작화면에 뜨는 그녀석들이 맞다

## Local Notification 알아보기

실제 구현에 앞서서 Local Notification이 무엇인지 알아보겠다.

Local 알림을 보내려면 알림 요청 즉 UNNotificationRequest를 작성해야한다.

여기서 UN은 User Notification의 약자이다. 이 UNNotificationRequest를 작성하려면 세 가지 요소가 필수적으로 필요하다. 먼저 **identifier(id)**와 **UNMutableNotificationContent(content)** 그리고 

**UNCalendarNotificationTrigger(달력 날짜 기준 트리거)** 

**UNTimeIntervalNotificationTrigger(시간 기준 트리거)**

**UNLocationNotificationTrigger(장소,위치에 따라 쓰는 트리거)** 

**(Trigger)** 

이 세 가지이다. 딱 봐도 아이디와 내용 그리고 트리거 이 세 가지이다. 

이 세 가지 요소가 모두 갖추어지면 UNNotificationRequest를 작성할 모든 준비가 끝난 것이다.

만들어진 UNNotificationRequest를 UNNotificationCenter에 보관하고

적절한 순간(trigger)에 알림으로 보내진다.

이제 진짜 앱을 만들어보자!

항상 하던대로 UI부터 구성해보겠다.

기본 ViewController 파일 대신 UITableViewController를 상속한 AlertListViewController를 만들어준다.

메인 스토리보드에서 NavigationController를 추가한 후, 연결된 root viewController에 클래스를 AlertListViewController를 추가한다. 

NavigationController에 Prefers Large Title 추가

TableView의 스타일은 Grouped, 그리고 바에 + 버튼을 추가해준다.

해당 작업이 끝난 후, 셀을 구성해줬다.
<img width="1792" alt="스크린샷 2023-12-29 오전 11 05 48" src="https://github.com/jinyongyun/Drink_Time_Alert_APP/assets/102133961/b9c49c9a-d136-4c58-9ffd-6c46cdd8e347">
당연히 Label이랑 outlet 연결해주고, switch는 valueChanged 액션 메서드도 추가한 뒤

storyboard에 alertListCell을 추가한다.

```swift
import UIKit

class AlertListViewController: UITableViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let nibName = UINib(nibName: "AlertListCell", bundle: nil)
        tableView.register(nibName, forCellReuseIdentifier: "AlertListCell")
    }
    
    @IBAction func addAlertButtonAction(_ sender: UIBarButtonItem) {
    }
}
```

이제 List에 추가되는 alert 객체를 만들어보자

```swift
//
//  Alert.swift
//  Drink
//
//  Created by jinyong yun on 12/29/23.
//

import Foundation

struct Alert: Codable {
    var id: String = UUID().uuidString
    let date: Date
    let isOn: Bool
    
    var time: String {
        let timeFormatter = DateFormatter()
        timeFormatter.dateFormat = "hh:mm"
        return timeFormatter.string(from: date)
    }
    
    var meridiem: String {
        let meridiemFormatter = DateFormatter()
        meridiemFormatter.dateFormat = "a"
        meridiemFormatter.locale = Locale(identifier: "ko")
        return meridiemFormatter.string(from: date)
    }
}
```

time과 meridiem 변수를 추가해(time과 오전 오후 보다 편하게 사용하기 위해) alert에 설정된 date로부터 시간과 오전 오후를 리턴하도록 했다. (이때 오전오후 나타내는 포맷터에는 locale을 꼭 한국으로 설정해줘야한다!!)

AlertListViewController에 **var** alertList: [Alert] = [] 다음과 같은 빈 Alert 배열을 추가해주고

> Command + control + e 하면 모든 변수명 수정, alert → alertList로 수정해줬다.
> 

AlertListViewController에 DataSource와 Delegate를 추가해준다.

앞에서 추가해 준 alertList의 count 즉 길이만큼 테이블뷰에 나타나게 해주고, titleForHeaderInSection 메서드를 이용해 Section에 따라 나타내는 문구를 다르게 해준다.

cellForRowAt에 dequeuReusableCell을 이용하여 cell을 인스턴스화 해주고, 해당 셀의 outlet에 alertList의 (우리가 선택한) 멤버변수를 이용해 넣어준다.

```swift
//UITableView DataSource, Delegate
extension AlertListViewController {
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return alertList.count
    }
    
    override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        switch section {
        case 0:
            return "🚰 물마실 시간"
        default:
            return nil
        }
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "AlertListCell", for: indexPath) as? AlertListCell else { return UITableViewCell() }
        cell.alertSwitch.isOn = alertList[indexPath.row].isOn
        cell.timeLabel.text = alertList[indexPath.row].time
        cell.meridiemLabel.text = alertList[indexPath.row].meridiem
        
        return cell
    }
    
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 80
    }
}
```

그리고 추후에 알림을 삭제하거나 셀을 삭제할 때, 알림 전체가 삭제될 수 있게 액션을 추가할 것이다. 즉 델리게이트 추가!

canEditRowAt 메서드를 이용해 셀을 수정할 수 있도록 바꿔주고 

```swift
override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath){
        switch editingStyle {
        case .delete:
            //notification 삭제 구현
            return
        default:
            break
        }
    }
```

editingStyle 메서드를 이용해 editingStyle이 .delete시 동작을 여기에 적어주면 되는데, 이건 나중에 구현하도록 하자

그 다음 단계는 + 버튼을 눌렀을 때, 전환될 AddAlertViewController를 만드는 것이다. Alert을 추가하기 위한 ViewController이다.
<img width="1792" alt="스크린샷 2023-12-29 오후 2 51 48" src="https://github.com/jinyongyun/Drink_Time_Alert_APP/assets/102133961/6feb76de-465f-47c1-9183-5e49f003eb13">

화면 구성은 다음과 같이 해주었고, AddAlertViewController에 IBOutlet과 취소, 저장 바 버튼을 눌렀을 때의 Action 메서드도 만들어줬다. 이름과 같이 취소를 누르면 화면 자체가 dismiss 되고, 저장을 누르면 새로운 알람이 만들어질 것이다.  save 버튼을 눌렀을 때, 우리가 datePicker에서 눌렀던 시간을 부모 뷰에 전달해줘야 리스트에 표현이 될 것이다. 따라서 저장 버튼을 누르면 설정한 시간값이 부모뷰에 전달될 수 있도록 클로져를 통해서 전달해보겠다.

```swift
import UIKit

class AddAlertViewController: UIViewController {
    
    var pickedDate: ((_ date:Date) -> Void)?
    
    
    @IBOutlet weak var datePicker: UIDatePicker!
    
    
    @IBAction func dismissButtonTapped(_ sender: UIBarButtonItem) {
        self.dismiss(animated: true, completion: nil)
    }
    
    @IBAction func saveButtonTapped(_ sender: UIBarButtonItem) {
        pickedDate?(datePicker.date)
        self.dismiss(animated: true, completion: nil)
    }
    
    
}
```

다음과 같이 pickedDate 클로저를 선언해주고(있을 지 없을 지 모르니까 옵셔널) 해당 클로저에서는 date를 받아서 저장한다.

이걸 saveButtonTapped에 넣어주면 끝

이제 ListView에서 + 버튼을 누르면 이 AddAlertViewController가 표현되도록 하기 위해 AlertListViewController에 만들어 둔 addAlertButtonAction 메서드에 클로저를 구현할 것이다.

```swift
@IBAction func addAlertButtonAction(_ sender: UIBarButtonItem) {
        guard let addAlertVC = storyboard?.instantiateViewController(withIdentifier: "AddAlertViewController") as? AddAlertViewController else { return }
        self.present(addAlertVC, animated: true, completion: nil)

        addAlertVC.pickedDate = { [weak self] date in
            guard let self = self else { return }
            
            let newAlert = Alert(date: date, isOn: true)
            
        }
    }
```

addAlertViewController를 인스턴스화 해주고, addAlertViewController의 pickedDate 클로저를 구현해준다. 순환참조방지를 위해 [weak self] 넣어주고, 새로운 Alert을 만들어준다.

그리고 앱을 껐다 켜도 해당 정보가 유지되게 하기 위해 UserDefaults에 해당 정보들을 저장하는 시스템을 만들건데, 이와 관련된 함수를 만들어줘야한다.

함수 이름을 alertList로 해줄거라, 기존 alert 배열 alerts로 이름 변경!

```swift
func alertList() -> [Alert] { // alertList 메서드는 Alert 배열을 UserDefaults에서 내뱉어준다!
        guard let data = UserDefaults.standard.value(forKey: "alerts") as? Data,
              let alerts = try? PropertyListDecoder().decode([Alert].self, from: data) else { return [] }
        return alerts
    }
```

UserDefaults는 우리가 임의로 만든 custom한 리스트를 읽을 땐 (UserDefaults가 이해 못함, 이녀석은 바보라 제한적인 타입만 이해, Boolean이나 String 등은 이해) 프로퍼티를 decode 해줘야 읽을 수 있다. 따라서 PropertyListDecoder로 디코드 해주고, 받은 Alert 배열을 리턴해준다.

만든 alertList 함수를 viewWillAppear에서 실행시켜 alerts에 넣어주면 된다.

```swift
@IBAction func addAlertButtonAction(_ sender: UIBarButtonItem) {
        guard let addAlertVC = storyboard?.instantiateViewController(withIdentifier: "AddAlertViewController") as? AddAlertViewController else { return }
        
        addAlertVC.pickedDate = { [weak self] date in
            guard let self = self else { return }
            
            var alertList = self.alertList()
            
            let newAlert = Alert(date: date, isOn: true)
            
            alertList.append(newAlert)
            
            alertList.sort {$0.date < $1.date}
            
            self.alerts = alertList
            
            UserDefaults.standard.set(try? PropertyListEncoder().encode(self.alerts), forKey: "alerts")
            
            self.tableView.reloadData()
        }
        self.present(addAlertVC, animated: true, completion: nil)
    }
```

당연히 새로운 Alert을 생성해주는 addAlertButtonAction에서도 alertList (UserDefaults에서 Alert리스트 뱉어주는 함수) 에서 받아온 alertList에 새로 만든 Alert을 추가해주고 이녀석을 날짜순으로 정렬한다음, 다시 alerts에 넣어준다. (아까 이름 바꿔준 그녀석) 그리고 UserDefaults에 해당 alerts를 다시 저장해줘야하는데 이때도 우리가 Encode 해줘야된다. 마지막으로 reloadData 해주면 끗!!

이제 위에서 비워놓았던 editingStyle이 .delete 일 때 부분을 채워줘야한다(쉽게 말해 셀 지웠을 때)

```swift
 override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath){
        switch editingStyle {
        case .delete:
            //notification 삭제 구현
            self.alerts.remove(at: indexPath.row)
            UserDefaults.standard.set(try? PropertyListEncoder().encode(self.alerts), forKey: "alerts")
            self.tableView.reloadData()
            return
        default:
            break
        }
    }
```

로직은 간단!

alerts 에서 지워주고, 다시 UserDefaults에서 업데이트 해주면 끝이다. (마지막엔 언제나 리로드)

마지막에 마지막으로 UISwitch 값을 마찬가지로 저장해줘야 하는데, 이녀슥은 AlertListCell에 있는데(alertSwitchValueChanged 액션 메서드로 연결해놓았다) 어떻게 각 cell에서 index값을 알 수 있게 할까?

바로 AlertListViewController에서 

**cell.alertSwitch.tag = indexPath.row** 

다음으르 추가하여, 각 alert의 alertSwitch의 tag에 indexPath.row를 지정하는 것이다.(태그사용)

```swift
@IBAction func alertSwitchValueChanged(_ sender: UISwitch) {
        guard let data = UserDefaults.standard.value(forKey: "alerts") as? Data,
                var alerts = try? PropertyListDecoder().decode([Alert].self, from: data) else {return}
        alerts[sender.tag].isOn = sender.isOn
        UserDefaults.standard.set(try? PropertyListEncoder().encode(alerts), forKey: "alerts")
    }
```

로직은 아까와 비슷…UserDefaults에서 alerts 받아와서 (물론 디코드)

태그를 이용해 indexPath 접근 후, isOn으로 바꿔주고

다시 userDefaults에 저장

그렇게 완성된 모든 UI
<img width="1792" alt="스크린샷 2023-12-29 오후 4 02 46" src="https://github.com/jinyongyun/Drink_Time_Alert_APP/assets/102133961/ef5e631c-edfb-45d4-9057-25bc01f3e697">



https://github.com/jinyongyun/Drink_Time_Alert_APP/assets/102133961/3015bf77-2f34-4c1a-9839-bbc56fa8898a

## Content 설정하기

알림을 만들면 앱에서 어떤 형태로 표현할 것인지를 표시할 수  있는데, 이런 내용을 설정하기 위해 AppDelegate에서  설정해보겠다. AppDelegate에서 NotificationCenter를 import 해주고

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        
        UNUserNotificationCenter.current().delegate = self
        return true
    }
```

didFinishLaunchingWithOptions 메서드에서 delegate 설정을 해준다. 그럼 delegate를 구현해줘야하는데 깔끔하게 extension으로 빼서 작성해주자

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .list, .badge, .sound])
    }
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        completionHandler()
    }
}
```

다음과 같이 UNUserNotificationCenterDelegate에는 구현할 수 있는 메서드 2개가 있는데

하나는 willPresent , 즉 NotificationCenter 보내기 전에 어떤 핸들링을 해줄 것인가.

여기서는 각각의 알림이 banner와 list, badge, sound를 포함하도록 설정해줬고

만약 소리가 없는 알람을 보내고 싶다거나, 배너에 표시하고 싶지 않다면 여기서 설정해주면 된다.

마지막은 didReceive 보낸 후 어떤 핸들링 , 여기서는 그냥 completionHandler 적어주고 넘어갔다. 

특정 앱에서 로컬 알림을 포함한 알림을 받으려면 사용자의 승인을 받아야 한다.

따라서 사용자의 승인을 구하는 코드를 추가한다.

```swift
import UIKit
import NotificationCenter
import UserNotifications

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var userNotificationCenter = UNUserNotificationCenter.current()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        
        UNUserNotificationCenter.current().delegate = self
        
        let authorizationOptions = UNAuthorizationOptions(arrayLiteral: [.alert, .badge, .sound])
        userNotificationCenter.requestAuthorization(options: authorizationOptions) { _, error in
            if let error = error {
                print("ERROR: notification authorization request \(error.localizedDescription)")
            }
        }
        return true
    }
```

UserNotification을 import 해주고, UNAuthorizationOptions를 통해 권한을 물어볼 옵션(어떤 권한에 대해 요청할 것인가?)를 설정한 다음

이 옵션을 userNotificationcenter의 requestAuthorization 메서드의 옵션 매개변수에 넣어준다. 그럼 물어보겠지?

핸들러로는 간단하게 에러만 처리하도록 해주었다.

우리의 샘플 앱에서 로컬 알림이 설정되는 곳은 어디일까?? 먼저 AlertListViewController에서 DatePickerClosure를 통해서 알람이 추가될 때가 될 것이다.

하나 더 있는데 AlertListCell 부분에서 switch가 켜질 때가 될 것이다.

그래서 UNUserNotificationCenter에 범용함수를 만들어 Alert 객체를 받아 request를 만들고 최종적으로 NotificationCenter에 추가해야한다.

```swift
//
//  UNNotificationCenter.swift
//  Drink
//
//  Created by jinyong yun on 12/30/23.
//

import Foundation
import UserNotifications

extension UNUserNotificationCenter {
    func addNotificationRequest(by alert: Alert) {
        let content = UNMutableNotificationContent()
        content.title = "물 마실 시간이예요! 🚰"
        content.body = "세계보건기구(WHO)가 권장하는 하루 물 섭취량은 1.5~2L 입니다."
        content.sound = .default
        content.badge = 1
    }
    
    
}
```

그래서 다음과 같이 UNUserNotificationCenter 파일을 만들고 extension으로 추가적인 범용함수를 만들었다. 이녀석의 역할은 위해서 말했듯이 Alert 객체를 받아 request를 만들고 최종적으로 NotificationCenter에 추가하는 함수이다. by alert을 통해 alert에 의해 동작하는 함수임을 알리고, UNMutableNotificationContent() 메서드로 content를 만든다. title, body, sound, badge를 각각 설정해주는데

이때 badge를 설정한 뒤, 자동적으로 이 badge가 없어지는 것이 아니기 때문에, 우리가 직접 언제 없어져야 하는 지 설정해줘야 한다.

가장 적절한 위치는 SceneDelegate의 sceneDidBecomeActive 메서드일 것이다. 사용자가 앱을 열어 active 상태가 되었을 때 뱃지를 없애주는 것이다. 이 흐름이 가장 자연스러울 것이다.

```swift
func sceneDidBecomeActive(_ scene: UIScene) {
       // UIApplication.shared.applicationIconBadgeNumber = 0 // applicationIconBadgeNumber was deprecated in iOS 17.0
        UNUserNotificationCenter.current().setBadgeCount(0)
    }
```

UIApplication.shared.applicationIconBadgeNumber는 IOS 17에서 deprecated 되었다고 해서,,,

UNUserNotificationCenter.current().setBadgeCount(0) 를 대신 사용했다.

[setBadgeCount(_:withCompletionHandler:) | Apple Developer Documentation](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/4031317-setbadgecount)

이렇게 하면 앱이 active 상태가 되었을 때, 뱃지가 사라질 것이다.

## Trigger 설정하기

이제 LocalNotification을 발송하는 조건이 되는 트리거를 설정해 볼 것이다. 우리는 DatePicker로 선택한 시간, 즉 AddAlertViewController의 pickedDate에 설정한 시간에 발송을 할 것이다.

특정 시간에 발송을 하는 것은 UNCalendarNotificationTrigger를 통해 구현할 수 있다.

```swift
//
//  UNNotificationCenter.swift
//  Drink
//
//  Created by jinyong yun on 12/30/23.
//

import Foundation
import UserNotifications

extension UNUserNotificationCenter {
    func addNotificationRequest(by alert: Alert) {
        let content = UNMutableNotificationContent()
        content.title = "물 마실 시간이예요! 🚰"
        content.body = "세계보건기구(WHO)가 권장하는 하루 물 섭취량은 1.5~2L 입니다."
        content.sound = .default
        content.badge = 1
        
        let component = Calendar.current.dateComponents([.hour, .minute], from: alert.date)
        let trigger = UNCalendarNotificationTrigger(dateMatching: component, repeats: alert.isOn)
    }
    
    
}
```

Calendar 클래스의 dateComponents를 활용해 alert.date에서 hour와 minute만 쏙 빼와서

해당 component를 UNCalendarNotificationTrigger에 추가한다.

repeat 즉 반복은 alert이 isON 되어있을 때 계속 반복해주는 것으로 했다.

## Request 설정하기

```swift
//
//  UNNotificationCenter.swift
//  Drink
//
//  Created by jinyong yun on 12/30/23.
//

import Foundation
import UserNotifications

extension UNUserNotificationCenter {
    func addNotificationRequest(by alert: Alert) {
        let content = UNMutableNotificationContent()
        content.title = "물 마실 시간이예요! 🚰"
        content.body = "세계보건기구(WHO)가 권장하는 하루 물 섭취량은 1.5~2L 입니다."
        content.sound = .default
        content.badge = 1
        
        let component = Calendar.current.dateComponents([.hour, .minute], from: alert.date)
        let trigger = UNCalendarNotificationTrigger(dateMatching: component, repeats: alert.isOn)
        
        let request = UNNotificationRequest(identifier: alert.id, content: content, trigger: trigger)
        self.add(request, withCompletionHandler: nil) //자기 자신 즉 UNUserNotificationCenter에 추가 
   }
    
    
}
```

request는 간단하다 앞서 만들어둔 identifier(alert의 id를 만든 이유) , content, trigger를 각각 넣어주면 된다.

이제 문제는 alert을 만들어주는 두 부분에 addNotificationRequest를 추가해주는 것이다.

AlertListViewController에 **import** UserNotifications

**let** userNotificationCenter = UNUserNotificationCenter.current()

추가

```swift
@IBAction func addAlertButtonAction(_ sender: UIBarButtonItem) {
        guard let addAlertVC = storyboard?.instantiateViewController(withIdentifier: "AddAlertViewController") as? AddAlertViewController else { return }
        
        addAlertVC.pickedDate = { [weak self] date in
            guard let self = self else { return }
            
            var alertList = self.alertList()
            
            let newAlert = Alert(date: date, isOn: true)
            
            alertList.append(newAlert)
            
            alertList.sort {$0.date < $1.date}
            
            self.alerts = alertList
            
            UserDefaults.standard.set(try? PropertyListEncoder().encode(self.alerts), forKey: "alerts")
            
            //여기에 추가!!!
            self.userNotificationCenter.addNotificationRequest(by: newAlert)
            
            self.tableView.reloadData()
        }
        self.present(addAlertVC, animated: true, completion: nil)
    }
```

AlertListCell도 

**import** UserNotifications

**let** userNotificationCenter = UNUserNotificationCenter.current() 추가

```swift
@IBAction func alertSwitchValueChanged(_ sender: UISwitch) {
        guard let data = UserDefaults.standard.value(forKey: "alerts") as? Data,
                var alerts = try? PropertyListDecoder().decode([Alert].self, from: data) else {return}
        alerts[sender.tag].isOn = sender.isOn
        UserDefaults.standard.set(try? PropertyListEncoder().encode(alerts), forKey: "alerts")
        
        if sender.isOn {
            userNotificationCenter.addNotificationRequest(by: alerts[sender.tag])
        }
    }
```

마지막에 sender가 on 되는 상태일 때마다 userNotificationCenter에 request 추가

그리고 셀을 삭제할 때 알림도 삭제해야하니까

```swift

//AlertListViewController
override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath){
        switch editingStyle {
        case .delete:
            //notification 삭제 구현
            self.alerts.remove(at: indexPath.row)
            UserDefaults.standard.set(try? PropertyListEncoder().encode(self.alerts), forKey: "alerts")
            
            //userNotificationCenter에 남아있는 request 중 id에 해당하는 request 삭제
            userNotificationCenter.removePendingNotificationRequests(withIdentifiers: [alerts[indexPath.row].id])
            
            self.tableView.reloadData()
            return
        default:
            break
        }

...
//AlertListCell
@IBAction func alertSwitchValueChanged(_ sender: UISwitch) {
        guard let data = UserDefaults.standard.value(forKey: "alerts") as? Data,
                var alerts = try? PropertyListDecoder().decode([Alert].self, from: data) else {return}
        alerts[sender.tag].isOn = sender.isOn
        UserDefaults.standard.set(try? PropertyListEncoder().encode(alerts), forKey: "alerts")
        
        if sender.isOn {
            userNotificationCenter.addNotificationRequest(by: alerts[sender.tag])
        }else{ // else만 추가하면 된다
            userNotificationCenter.removePendingNotificationRequests(withIdentifiers: [alerts[sender.tag].id])
        }
    }
```

0:49초에 알림이 옵니다!



https://github.com/jinyongyun/Drink_Time_Alert_APP/assets/102133961/7fd45cf6-896e-4d62-814b-9bf65eff769c


