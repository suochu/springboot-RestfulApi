
## 使用 springboot 開發 RESTful API 
1. 什麼是 API？
     - 使用API過程如同點菜(Input)、實作過程(Call API )、出餐點(Output)
     - 設計好API，前端可以透過Call API，獲取來自server資料庫的資料，並將資料render到頁面上
    
2. 什麼是 REST？
     - REST的無狀態特性是指後端server不會保留前端Http Request的狀態
     - 有狀態例子，例如：servlet A保留前端Http Request的狀態並forward到servlet B去
     - 思考有狀態有什麼的壞處？
      
5. 什麼是 RESTful API特性？
     - 無狀態、前後端分離
     - 前後端分離會碰什麼問題？
     
      
7. RESTful API 請求資源的方法
    - 是指基於Http協定創造方法與Server溝通要向資料庫進行哪些資料操作
    - Get request：獲取資料庫資料，常用於資料查詢
    - Post request：新增資料庫資料
    - Put request：資料庫資料沒資料直接新增的修改
    - Patch request：修改資料庫部分欄位資料
    - Delete request:刪除資料庫資料


9. 回應狀態碼


11. 開發上差異
12. 如何實作 
13. 開發思考路徑：
    - 1.建立資料表

    ![](https://hackmd.io/_uploads/BJXqVdNE3.png)

    - 2.實踐ORM，撰寫對應資料表欄位的model entity，entity交由繼承JpaRepository的Dao進行處理
    
    ```java=
	@SuperBuilder
    @NoArgsConstructor
    @Data
    @ToString(exclude = {"beverageOrders"})
    @Entity
    @Table(name = "BEVERAGE_GOODS", schema="LOCAL")
    public class BeverageGoods {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BEVERAGE_GOODS_SEQ_GEN")
    @SequenceGenerator(name = "BEVERAGE_GOODS_SEQ_GEN", sequenceName = "BEVERAGE_GOODS_SEQ", allocationSize = 1)
	@Column(name = "GOODS_ID")	
	private long goodsID;
	
	@Column(name = "GOODS_NAME")	
	private String goodsName;	
	
	@Column(name = "DESCRIPTION")	
	private String description;	
		
	@Column(name = "PRICE")	
	private long price;	
	
	@Column(name = "QUANTITY")	
	private long quantity;	
		
	@Column(name = "IMAGE_NAME")	
	private String imageName;	
	
		
	@Column(name = "STATUS")	
	private String status;
    
        // 雙向一對多關係(一與多的物件各自擁有對方的entity參照)
    @OneToMany(
        
	    // mappedBy連結雙向關係，所對映的物件欄位名稱
		mappedBy = "beverageGoods",
		cascade = {CascadeType.ALL}, orphanRemoval = true,
		fetch = FetchType.LAZY 
			
	)
		
	    //order為是多的一方
	    private List<BeverageOrder> beverageOrders;
        
	}
    ```
    - 3.撰寫controller，選定http request method，確立input vo的規格以及請求內容的類型
    ```java=
    @RestController
    @RequestMapping("/ecommerce/BackendController")
    public class BackendController {
    
    @Autowired
	private CriteriaQueryDao criteriaQueryDao;
	
	
	@Autowired
	private BackendService backendService;
	
	    //vo轉進enetity
	    @PostMapping(value = "/createGoods", consumes = { MediaType.MULTIPART_FORM_DATA_VALUE })
	    public ResponseEntity<BeverageGoods> createGoods(@ModelAttribute GoodsVo goodsVo) throws IOException {
        
		BeverageGoods goods = backendService.createGoods(goodsVo);
		
		return ResponseEntity.ok(goods);
	
	    }
    ```	
        
     - 4.撰寫service處理商業邏輯，request的vo轉換成與資料庫有關連的entity物件，交由dao進行持久化操作，將資料更新至資料庫
    
    ```java=
    @Service
    public class BackendService {

	@Autowired
	private BackendDao backendDao;
   
    //vo存進資料庫
	public BeverageGoods createGoods(GoodsVo goodsVo) throws IOException {
	// 複制檔案
	MultipartFile file = goodsVo.getFile();
	String fileName = goodsVo.getImageName();
	//把圖片複製到symbolic的目的地
	Files.copy(file.getInputStream(), Paths.get("/Users/suyongquan/Desktop/SuGoodsImg").resolve(fileName));		
	//意識到beverageGoods塞入商品需求時，GoodsID是自增主鍵值而不是由vo給的
		BeverageGoods beverageGoods=BeverageGoods.builder()
		//.goodsID(goodsVo.getGoodsID())
		.goodsName(goodsVo.getGoodsName())
			.description(goodsVo.getDescription())
			.price(goodsVo.getPrice())	
			.quantity(goodsVo.getQuantity())	
			.imageName(goodsVo.getImageName())
			.status(goodsVo.getStatus())
			.build();
            //backendDao.save(beverageGoods);持久化操作,才有將資料更新至資料庫的動作
			BeverageGoods goods=backendDao.save(beverageGoods);
			return goods;
			
    }
    ```
    
    - 5.Dao繼承JpaRepository
    ```java=
    @Repository
    public interface BackendDao extends JpaRepository<BeverageGoods, Long>{
        
	    BeverageGoods findByGoodsID(long goodsID);	
        
    }
    ```
    - 6.Springboot開發與Java web servlet+dao處理資料差別在哪？
        - Springboot開發將JDBC連線設定交由yml設定檔處理
        - 原先Dao透過ResultSet讀取對應資料庫資料，改由vo與entity處理
        - 簡易的SQL可由JPA內建方法調用
        - Java web servlet + dao
        
    ```java=
    public List<Goods> queryGoods(){
		
        List<Goods> goods = new ArrayList<>();
		// querySQL SQL
		String querySQL = "SELECT * FROM BEVERAGE_GOODS";
		// Step1:取得Connection
		try (Connection conn = DBConnectionFactory.getOracleDBConnection();
		    // Step2:Create prepareStatement For SQL
			PreparedStatement stmt = conn.prepareStatement(querySQL);
			ResultSet rs = stmt.executeQuery()){
			while(rs.next()){
				BigDecimal goods_ID = rs.getBigDecimal("goods_ID");
				String goods_Name = rs.getString("goods_Name");
				Integer Price = rs.getInt("Price");
				Integer Quantity = rs.getInt("Quantity");
				String Image_Name = rs.getString("Image_Name");
				String status = rs.getString("status");
				
				Goods good = new Goods();
				good.setGoodsID(goods_ID);
				good.setGoodsName(goods_Name);
				good.setGoodsPrice(Price);
				good.setGoodsQuantity(Quantity);
				good.setImageName(Image_Name);
				good.setStatus(status);
				//打包完物件,放進陣列在回傳
				goods.add(good);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		
		return goods;
	}
	```
    
