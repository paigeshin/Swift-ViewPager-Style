# Swift-ViewPager-Style

### Preparation

1. Main class which contains two collection view 
    - one collection view - **Tab**
        - Scroll Direction: Horizontal
        - Estimate Size: None
    - second collection view  - **Page**
        - Scroll Direction: Horizontal
        - Estimate Size: None
        - Paging Enabled
2. Two Cells for Main Class
    - Tab Menu Cell
    - Page Cell which holds one collection view
3. Page Cell with Collection View and two cells
    - One Cell For Content - **Content**
    - One Cell For Loading ****
    - Estimate Size: None

# Code

### Constants

```swift
struct K {
	static let LOADING_CELL_IDENTIFIER = "LoadingCC"
	static let PROFILE_PAGE_CC = "ProfilePageCC"
	static let PROFILE_LETTER_IDENTIFIER = "ProfileLetterCC"
}
```

### Loading Cell

```swift
//
//  LoadingCC.swift
//  Secret Letter
//
//  Created by innertainment on 2021/05/31.
//

import UIKit

class LoadingCC: UICollectionViewCell {
    
    @IBOutlet private weak var activityIndicator: UIActivityIndicatorView!
    
    func start() {
        activityIndicator.startAnimating()
    }
    
}
```

### Content Cell

```swift
//
//  ProfileLetterCC.swift
//  Secret Letter
//
//  Created by innertainment on 2021/06/08.
//

import UIKit

class ProfileLetterCC: UICollectionViewCell {

    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }

}
```

### Page Cell (Pagination Applied)

```swift
//
//  ProfilePageCC.swift
//  Secret Letter
//
//  Created by innertainment on 2021/06/08.
//

import UIKit

class ProfilePageCC: UICollectionViewCell {

    private let TAG = "ProfilePageCC"
    
    //Vertical CollectionView, Estimate Size None
    @IBOutlet private weak var collectionView: UICollectionView!
    
    private let refreshControl: UIRefreshControl = UIRefreshControl()
    
    //MARK: - Pagination
    private var nextURL: String?
    private var isPaging = false
    private var dataSource = ["a", "b", "c", "a", "b", "c", "a", "b", "c"]
    
    private enum CellType: Int {
        case Display = 0
        case Loading = 1
    }
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        collectionView.showsHorizontalScrollIndicator = false
        collectionView.showsVerticalScrollIndicator = false
        
        attachPageRefresher()
        
        collectionView.register(UINib(nibName: K.PROFILE_LETTER_IDENTIFIER, bundle: nil), forCellWithReuseIdentifier: K.PROFILE_LETTER_IDENTIFIER)
        collectionView.register(UINib(nibName: K.LOADING_CELL_IDENTIFIER, bundle: nil), forCellWithReuseIdentifier: K.LOADING_CELL_IDENTIFIER)
        collectionView.delegate = self
        collectionView.dataSource = self
        
    }

}

//MARK: - Refresh
extension ProfilePageCC {
    
    private func attachPageRefresher() {
        refreshControl.attributedTitle = NSAttributedString(string: "데이터 업데이트".localized())
        refreshControl.addTarget(self, action: #selector(self.refresh(_:)), for: .valueChanged)
        collectionView.refreshControl = refreshControl
        collectionView.addSubview(refreshControl) // not required when using UITableViewController
    }
    
    @objc func refresh(_ sender: UIRefreshControl) {
        paging(showLoading: false, refresh: true)
        refreshControl.endRefreshing()
    }

}

//MARK: - CollectionView
extension ProfilePageCC: UICollectionViewDelegate, UICollectionViewDataSource, UICollectionViewDelegateFlowLayout {
    
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 2
    }
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        switch section {
        case CellType.Display.rawValue:
            return dataSource.count
        case CellType.Loading.rawValue:
            return 1
        default:
            return 0
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        switch indexPath.section {
        case CellType.Display.rawValue:
            #warning("실제 데이터 보여주는 영역")
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: K.PROFILE_LETTER_IDENTIFIER, for: indexPath) as! ProfileLetterCC
            return cell
        case CellType.Loading.rawValue:
            //Loading
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: K.LOADING_CELL_IDENTIFIER, for: indexPath) as! LoadingCC
            cell.start()
            return cell
        default:
            return UICollectionViewCell()
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let topMargin: CGFloat = 21
        let bottomMargin: CGFloat = 21
        let topLabelHeight: CGFloat = 15
        let bottomLabelHeight: CGFloat = 15
        let spacing: CGFloat = 7
        return CGSize(width: collectionView.frame.width, height: topMargin + topLabelHeight + spacing + bottomLabelHeight + bottomMargin)
    }
    
}

//MARK: Pagination
extension ProfilePageCC {
    
    //Paging
    // @param delay {Dobule = 1}: 가장 아래 부분에 데이터 로드하는 표기를 얼마만큼 보여줄지 정해준다.
    // @param nextURL {String? = nil}: nextURL 값을 넣어주면 페이징을 한다는 것이고, 넣어주지 않으면 paging을 진행하지 않는다는 의미다.
    // @param showLoading {Bool = false}: 가장 아래 부분에 데이터 로드하는 표기를 보여줄지 안보여줄지 정해준다. refresh할 때는 보여주지 않는 것이 자연스럽다.
    // @param refresh {Bool = false}: refresh가 true에는 fetchLetterList()를 호출했을 때 결과 값을 그대로 assign 해준다. 반대로 refresh가 false에는 dataSource에 값을 append 해준다.
    public func paging(delay: Double = 1, nextURL: String? = nil, showLoading: Bool = false, refresh: Bool = false) {
        //paging 동시에 두 번 못하게 막기
        if self.isPaging {
            print(TAG, ":: => Already Paging")
            return
        }
        
        if showLoading {
            DispatchQueue.main.async {
                self.collectionView.reloadSections(IndexSet(integer: CellType.Loading.rawValue))
             }
        }

        DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
            self.fetchLetterList(nextURL: nextURL, refresh: refresh)
            self.nextURL = nil
        }
    }

    func scrollViewDidScroll(_ scrollView: UIScrollView) {

        let offsetY = scrollView.contentOffset.y
        let contentHeight = scrollView.contentSize.height
        let height = scrollView.frame.height

        // 스크롤이 테이블 뷰 Offset의 끝에 가게 되면 다음 페이지를 호출
        if offsetY > contentHeight - height {
            // Paging with Network
            if !isPaging {
//                guard let nextURL = nextURL else { return }
                paging(nextURL: nextURL, showLoading: true)
                isPaging = true
            }
        }
    }
    
    private func fetchLetterList(nextURL: String? = nil, refresh: Bool = false) {
        
        let data = ["a", "b", "c", "d", "e"]
        
        if refresh {
            self.dataSource = data
        } else {
            self.dataSource.append(contentsOf: data)
        }
        
        //emptyview handling
//        if self.dataSource.count > 0 {
//               self.emptyDataView?.removeFromSuperview()
//           }
        
        self.collectionView.reloadData()
        self.isPaging = false
        
    }
    
}
```

### Main Class

```swift
//
//  UserProfileVC.swift
//  Secret Letter
//
//  Created by innertainment on 2021/06/07.
//

import UIKit

class UserProfileVC: BaseVC {
    @IBOutlet private weak var vcTitleLabel: UILabel!
    @IBOutlet private weak var scrollView: UIScrollView!
    @IBOutlet private weak var profileImageView: UIImageView!
    
    @IBOutlet private weak var ageGenderLabel: UILabel!
    @IBOutlet private weak var usernameLabel: UILabel!
    @IBOutlet private weak var receivedLetterCountLabel: UILabel!
    @IBOutlet private weak var sentLetterCountLabel: UILabel!
    @IBOutlet private weak var subscriptionStartDateLabel: UILabel!
    @IBOutlet private weak var subscriptionLeftDateLabel: UILabel!
    
    //Horizontal CollectionView, Estimate Size None
    @IBOutlet private weak var tabCollectionView: UICollectionView!
    
    //Horizontal CollectionView, Estimate Size None, Paging Enabled
    @IBOutlet private weak var contentCollectionView: UICollectionView!
    
    private var dataSource = [String]()
    
    //tab
    private var tabMenuList = [TabMenu]()
    private var previousSelectedIndex: Int?
    private func loadTabMenu() {
        if let _ = previousSelectedIndex {
            tabMenuList[previousSelectedIndex!].isSelected = true
        } else {
            //선택된 탭이 없었다면 0번의 탭을 선택해줌
            tabMenuList[0].isSelected = true
            previousSelectedIndex = 0
        }
        tabCollectionView?.reloadData()
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        scrollView.showsHorizontalScrollIndicator = false
        scrollView.showsVerticalScrollIndicator = false
        scrollView.contentInsetAdjustmentBehavior = .never
        
        contentCollectionView.delegate = self
        contentCollectionView.dataSource = self
        contentCollectionView.register(UINib(nibName: K.PROFILE_PAGE_CC, bundle: nil), forCellWithReuseIdentifier: K.PROFILE_PAGE_CC)
        contentCollectionView.isPagingEnabled = true
        
        tabCollectionView.delegate = self
        tabCollectionView.dataSource = self
        tabCollectionView.backgroundColor = .clear
        tabCollectionView.register(UINib(nibName: K.TAB_CELL_IDENTIFIER, bundle: nil), forCellWithReuseIdentifier: K.TAB_CELL_IDENTIFIER)
        tabMenuList = LocalDataProvider().fetchUserProfileTabMenuTitle()
        loadTabMenu()
        tabCollectionView.reloadData()
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        profileImageView.layer.cornerRadius = profileImageView.frame.height / 2
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        navigationController?.setNavigationBarHidden(false, animated: animated)
    }
    
    override func translate() {
        super.translate()
    }
    
}

extension UserProfileVC: UICollectionViewDelegate, UICollectionViewDataSource, UICollectionViewDelegateFlowLayout {
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return tabMenuList.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        switch collectionView {
        case tabCollectionView:
            let tab = collectionView.dequeueReusableCell(withReuseIdentifier: K.TAB_CELL_IDENTIFIER, for: indexPath) as! TabCC
            tab.tabMenu = tabMenuList[indexPath.row]
            tab.delegate = self
            tab.indexPath = indexPath
            tab.backgroundColor = .clear
            return tab
        case contentCollectionView:
            let page = collectionView.dequeueReusableCell(withReuseIdentifier: K.PROFILE_PAGE_CC, for: indexPath) as! ProfilePageCC
            //            let pageIndex = indexPath.row
            //            switch pageIndex {
            //            case 0:
            //            case 1:
            //            }
            
            return page
        default:
            return UICollectionViewCell()
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        if collectionView == tabCollectionView {
            return CGSize(width: UIScreen.main.bounds.width / CGFloat(tabMenuList.count), height: collectionView.frame.height)
        }
        return CGSize(width: collectionView.frame.width, height: collectionView.frame.height)
    }
    
    func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
        if contentCollectionView == scrollView {
            let index = Int(targetContentOffset.pointee.x / contentCollectionView.frame.width)
            selectPage(index: index)
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 0
    }
    
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
        return 0
    }
    
    private func selectPage(index: Int) {
        deselectAll()
        self.previousSelectedIndex = index //이전에 선택했던 탭 저장하기
        tabMenuList[index].isSelected = true
        tabCollectionView.reloadData()
    }
    
    private func deselectAll() {
        for i in 0...tabMenuList.count - 1 {
            tabMenuList[i].isSelected = false
        }
    }
    
}

extension UserProfileVC: TabCCDelegate {
    
    func onTabTapped(_ tabCC: TabCC, indexPath: IndexPath) {
        selectPage(index: indexPath.row)
        contentCollectionView.isPagingEnabled = false
        contentCollectionView.scrollToItem(at: indexPath, at: .centeredHorizontally, animated: true)
        contentCollectionView.isPagingEnabled = true
    }
    
}
```