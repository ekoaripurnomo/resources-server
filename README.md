* Create Resources Server
* open start.spring.io
* Generate SpringBoot Project use dependency web, jpa, thymleaf, lombok, mysqlcon
* persiapan memidahkan file java yang lalu ke resources server
* create package id.co.hanoman.latihan.resources.server, id.co.hanoman.latihan.resources.server.dao, id.co.hanoman.latihan.resources.server.controller
	id.co.hanoman.latihan.resources.server.entity, id.co.hanoman.latihan.resources.server.config
	
* create Alamat.java di package id.co.hanoman.latihan.resources.server.entity

	```java
	@Entity
	@Table(name="alamat")
	@Data
	public class Alamat {
		@Id
		@GeneratedValue(generator="uuid")
		@GenericGenerator(name = "uuid", strategy = "uuid2")
		private String id;
		
		@Column(nullable=false)
		private String jalan;
		
		@Column(nullable=false)
		private String kota;
		
		@Column(nullable=false)
		private String propinsi;
		
		@Column(nullable=false, length=5)
		private int kodepos;
	}
	```
	
* create User.java di package id.co.hanoman.latihan.resources.server.entity

	```java
	@Entity
	@Table(name="users")
	@Data
	public class User {
		@Id
		@GeneratedValue(strategy = GenerationType.AUTO)
		@Column(name = "id")
		private int id;
		
		@Column(name="username", unique=true, nullable=false)
		private String username;
		
		@Column(name="password", nullable=false)
		private String password;
		
		@Column(name="enabled", columnDefinition="tinyint(1) default 1")
		private boolean enabled;
	}
	```
	
* create Authority.java di package id.co.hanoman.latihan.resources.server.entity

	```java
	@Entity
	@Table(name="authorities")
	@Data
	public class Authority {
		@Id
		@GeneratedValue(strategy = GenerationType.AUTO)
		@Column(name="id")
		private int id;
		
		@ManyToOne
		@JoinColumn(name="id_user", nullable=false)
		private User idUser;
		
		@Column(name="authority", nullable=false)
		private String authority;
	}
	```
	
* Edit application.properties 
	
	```txt
	spring.datasource.url=jdbc:mysql://192.168.227.133:3306/latihan?useSSL=false
	spring.datasource.username=root
	spring.datasource.password=123456
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver
	spring.jpa.generate-ddl=true
	spring.jpa.show-sql=true
	spring.jpa.properties.hibernate.format_sql=true
	spring.jackson.serialization.indent_output=true
	spring.jpa.hibernate.ddl-auto=create

	project.base-dir=file:///D:/Data/Belajar/Java/spring-boot-latihan-resources-server
	spring.thymeleaf.prefix=${project.base-dir}/src/main/resources/templates/
	spring.thymeleaf.cache=false
	spring.resources.static-locations=${project.base-dir}/src/main/resources/static/
	spring.resources.cache-period=0
	```
	
* test gradle bootRun untuk melihat hasil table dari entity

* create import.sql on folder src/main/resources

	```sql
	insert into alamat (id, jalan, kota, propinsi, kodepos) values ('1b','Jalan Penggilingan','Cakung','Jakarta Timur',13940);
	insert into alamat (id, jalan, kota, propinsi, kodepos) values ('2b','Jalan Komarudin','Cakung','Jakarta Timur',13940);
	insert into alamat (id, jalan, kota, propinsi, kodepos) values ('3b','Jalan Duku','Kebon Jeruk','Jakarta Pusat',57890);
	insert into alamat (id, jalan, kota, propinsi, kodepos) values ('4b','Jalan Kelapa','Jagakarsa','Jakarta Selatan',23956);
	insert into alamat (id, jalan, kota, propinsi, kodepos) values ('5b','Jalan Patriot','Bekasi Barat','Bekasi',47863);

	insert into users (username, password, enabled) values ('eko','eko123',true);
	insert into users (username, password, enabled) values ('adi','adi123',true);
	insert into users (username, password, enabled) values ('edi','edi123',false);

	insert into authorities (id_user, authority) values ('1','Admin');
	insert into authorities (id_user, authority) values ('1','Operator');
	insert into authorities (id_user, authority) values ('2','Operator');
	insert into authorities (id_user, authority) values ('3','operator');
	```

* test gradle bootRun untuk melihat hasil inser import.sql ke table dari hasil entity
	
* create interface AlamatDao.java di package id.co.hanoman.latihan.resources.server.dao
	
	```java
	public interface AlamatDao {

	}
	```
	
	extends interface tsb dengan PagingAndSortingRepository
	
	```java
	public interface AlamatDao extends PagingAndSortingRepository<Alamat, String>{

	}
	```
	
* create RestController AlamatController.java di package id.co.hanoman.latihan.resources.server.controller

	```java
	@RestController
	@CrossOrigin
	@RequestMapping("/api")
	public class AlamatController {
		
		@Autowired
		private AlamatDao ad;
		
		@RequestMapping(value="/alamat", method=RequestMethod.GET)
		public Page<Alamat> daftarAlamat(Pageable page){
			return ad.findAll(page);
		}	
		
		@RequestMapping(value="/alamat", method=RequestMethod.POST)
		@ResponseStatus(HttpStatus.CREATED)
		public void insertAlamatBaru(@RequestBody @Valid Alamat a){
			ad.save(a);
		}	
		
		@RequestMapping(value="/alamat/{id}", method=RequestMethod.DELETE)
		@ResponseStatus(HttpStatus.OK)
		public void hapusAlamat(@PathVariable("id") String id){
			ad.delete(id);
		}
		
		@RequestMapping(value="/alamat/{id}", method=RequestMethod.PUT)
		@ResponseStatus(HttpStatus.OK)
		public void updateAlamat(@PathVariable("id") String id, @RequestBody @Valid Alamat a){
			a.setId(id);
			ad.save(a);
		}
		
		@RequestMapping(value="/alamat/{id}", method=RequestMethod.GET)
		@ResponseStatus(HttpStatus.OK)
		public ResponseEntity<Alamat> cariAlamatById(@PathVariable("id") String id){
			Alamat hasil = ad.findOne(id);
			if(hasil==null){
				return new ResponseEntity<>(HttpStatus.NOT_FOUND);
			}
			return new ResponseEntity<>(hasil, HttpStatus.OK);
		}
		
	}
	```

* test gradle bootRun, check http://localhost:8080/api/alamat

* add dependencies OAuth2 to build.gradle
	
	```txt
	compile('org.springframework.security.oauth:spring-security-oauth2')
	```

* create SecurityConfiguration.java for security resources server as backend on package id.co.hanoman.latihan.resources.server.config
	
	```java	
	public class SecurityConfiguration {

	}
	```
	
	add anotation and extends class :
	
	```java
	@EnableWebSecurity
	@EnableResourceServer
	public class SecurityConfiguration extends ResourceServerConfigurerAdapter{

	}
	```
	
	create method berikut untuk mengauthentifikasi request yang masuk :
	
	```java
	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().anyRequest().authenticated();
	}
	```
	
	Create method berikut untuk set Token Service :
	
	```java
	@Override
	public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
		RemoteTokenServices remoteToken = new RemoteTokenServices();
		remoteToken.setClientId("hanoman");
		remoteToken.setClientSecret("hanoman123");
		remoteToken.setCheckTokenEndpointUrl("http://localhost:10000/oauth/check_token");
		resources.tokenServices(remoteToken);
	}
	```
	
* untuk melihat debug, create debug=true pada SecurityConfiguration :

	```java
	@EnableWebSecurity(debug=true)
	@EnableResourceServer
	public class SecurityConfiguration extends ResourceServerConfigurerAdapter {
	...
	}
	```
	
* Lakukan perubahan pada project auth-server dbi :
	Authentication menggunakan password (grand_type=password)
	edit method configure(ClientDetailsServiceConfigurer clients) pada OAuth2Configuration.java
	
	```java
	.and().withClient("hci")
				.secret("hci123")
				.authorizedGrantTypes("password")
				.scopes("alamat")
				.authorities("Operator")
				.accessTokenValiditySeconds(600);
	```	
	
	tambahkan methode AuthenticationManager di class OAuth2Configuration.java
	
	letakan paling atas method
	
	```java
	@Autowired
	@Qualifier("authenticationManagerBean")
	private AuthenticationManager authenticationManager;
	```
	
	letakan paling bawah method
	
	```java
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		endpoints.authenticationManager(authenticationManager);
	}
	```
	
	
	tambahkan method berikut di SecurityConfiguration.java pada project auth-server 
	letakan paling bawah method
	
	```java
	@Override
	@Bean
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}
	```
	
* get token use RestAPI Console :
	```rest
	http://localhost:10000/oauth/token
	application/x-www-form-urlencoded
	Request Parameter:
		grant_type=password
		username=eko <-sesuai dengan username database
		password=eko123 <-sesuai dengan password username database
	Basic Auth:
		username=hci <- client_id
		password=hci123 <- client_secret
	Click POST :
	{
    "access_token": "de8cd638-b76b-45e2-aeaf-2bd36b48a0a7",
    "token_type": "bearer",
    "expires_in": 599,
    "scope": "alamat"
	}
	```
	
* Set PreAuthorize anotation pada Rest Controller (AlamatController.java):
	
	@PreAuthorize("hasAuthority('Operator')")
	@RequestMapping(value="/alamat", method=RequestMethod.GET)
	public Page<Alamat> daftarAlamat(Pageable page){
		return ad.findAll(page);
	}
	
* Authentication menggunakan implicit add script ke method configure(ClientDetailsServiceConfigurer clients) 
	di class OAuth2Configuration.java pada project auth-server
	
	```java
	.and().withClient("hciimp")
				.secret("hciimp123")
				.authorizedGrantTypes("implicit")
				.scopes("alamat")
				.authorities("Operator")
				.accessTokenValiditySeconds(600)
	```
* get token use url : http://localhost:10000/oauth/authorize?client_id=hciimp&response_type=token&redirect_uri=http://example.com
	result : http://example.com/#access_token=d494b6f1-a1c2-418e-84e0-6c47edd1e643&token_type=bearer&expires_in=599&scope=alamat
	
* check token langsung ke halaman api/alamat
	http://localhost:8080/api/alamat?access_token=d494b6f1-a1c2-418e-84e0-6c47edd1e643
	result berupa json format