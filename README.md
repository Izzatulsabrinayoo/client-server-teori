# client-server-teori
 Project Client Server Teori
Nama : Izzatul Sabrina
NoBP : 2101082008
Kelas : Tekom 2B


# Application Context
ApplicationContext adalah sebuah interface representasi container IoC di Spring dan inti dari Spring Framework.
ApplicationContext banyak sekali class implementasinya, secara garis besar dibagi menjadi 2 jenis implementasi, XML dan Annotation

Pada versi Spring 3, XML masih menjadi pilihan utama, namun sekarang sudah banyak orang beralih dari XML ke Annotation, bahkan Spring Boot pun merekomendasikan menggunakan Annotation untuk membuat aplikasi Spring
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html


# Configuration
Untuk membuat ApplicationContext menggunakan Annotation, pertama kita bisa perlu membuat Configuration class Configuration Class adalah sebuah class yang terdapat annotation @Configuration pada class tersebut

@Configuration
public class FarizConfiguration {
}

Membuat Application Context
Selanjutnya, setelah membuat Class Configuration, kita bisa menggunakan class AnnotationConfigApplicationContext untuk membuat Application Context

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html

public class ApplicationContextTest {
    @Test
    void testApplicationContext(){
        ApplicationContext context = new AnnotationConfigApplicationContext(FarizConfiguration.class);  
        Assertions.assertNotNull(context);
    }
}



# Singleton
Singleton adalah salah satu Design Patterns untuk pembuatan objek, dimana sebuah object hanya dibuat satu kali saja Dan ketika kita membutuhkan object tersebut, kita hanya akan menggunakan object yang sama

public class Database {
    private static Database database;
    public static Database getInstance(){
        if(database == null){
            database = new Database();
        }
        return database;
    }
    private Database(){   
    }
}
public class DatabaseTest {
    @Test
    void testSingleton(){
        var database1 = Database.getInstance();
        var database2 = Database.getInstance();
        Assertions.assertSame(database1, database2);
    }
}public class DatabaseTest {
    @Test
    void testSingleton(){
        var database1 = Database.getInstance();
        var database2 = Database.getInstance();
        Assertions.assertSame(database1, database2);
    }
}



# Bean
Secara default, bean merupakan singleton, artinya jika kita mengakses bean yang sama, maka dia akan mengembalikan object yang sama. Kita juga bisa mengubahnya jika tidak ingin singleton

Untuk membuat bean, kita bisa membuat sebuah method di dalam class Configuration Selanjutnya nama method tersebut akan menjadi nama bean nya, dan return object nya menjadi object bean nya Method tersebut perlu kita tambahkan annotation @Bean, untuk menandakan bahwa itu adalah bean Secara otomatis Spring akan mengeksekusi method tersebut, dan return value nya akan dijadikan object bean secara otomatis, dan disimpan di container IoC

@Slf4j
@Configuration
public class BeanConfiguration {
    @Bean
    public Foo foo(){
        Foo foo = new Foo();
        log.info("Create new foo");
        return foo;
    }
}
public class BeanTest {
    @Test
    void testCreateBean(){
        ApplicationContext context = new AnnotationConfigApplicationContext(BeanConfiguration.class); 
        Assertions.assertNotNull(context);
    }
    @Test
    void testGetBean(){
        ApplicationContext context = new AnnotationConfigApplicationContext(BeanConfiguration.class);  
        Foo foo1 = context.getBean(Foo.class);
        Foo foo2 = context.getBean(Foo.class);  
        Assertions.assertSame(foo1, foo2);
    }
}



# Duplicate Bean
Di Spring, kita bisa mendaftarkan beberapa bean dengan tipe yang sama Namun perlu diperhatikan, jika kita membuat bean dengan tipe data yang sama, maka kita harus menggunakan nama bean yang berbeda Selain itu, saat kita mengakses bean nya, kita wajib menyebutkan nama bean nya, karena jika tidak, Spring akan bingung harus mengakses bean yang mana

@Configuration
public class DuplicateConfiguration {
    @Bean
    public Foo foo1(){
        return new Foo();
    }    
    @Bean
    public Foo foo2(){
        return new Foo();
    }
}
public class DuplicateTest {   
    @Test
    void testDuplicate(){       
        ApplicationContext context = new AnnotationConfigApplicationContext(DuplicateConfiguration.class);       
        Assertions.assertThrows(NoUniqueBeanDefinitionException.class,() -> {
            Foo foo = context.getBean(Foo.class);
        });      
    }
}



# Primary Bean
Dengan memilih salah satunya menjadi primary, secara otomatis jika kita mengakses bean tanpa menyebutkan nama bean nya, secara otomatis primary nya yang akan dipilih Untuk memilih primary bean, kita bisa tambahkan annotaiton @Primary

@Configuration
public class PrimaryConfiguration {
    @Primary
    @Bean
    public Foo foo1(){
        return new Foo();
    }  
    @Bean
    public Foo foo2(){
        return new Foo();
    }
}
public class PrimaryTest {    
    private ApplicationContext applicationContext;    
    @BeforeEach
    void setup(){
        applicationContext = new AnnotationConfigApplicationContext(PrimaryConfiguration.class);
    }
    @Test
    void testGetPrimary(){
        Foo foo = applicationContext.getBean(Foo.class);
        Foo foo1 = applicationContext.getBean("foo1",Foo.class);
        Foo foo2 = applicationContext.getBean("foo2",Foo.class);      
        Assertions.assertSame(foo, foo1);
        Assertions.assertNotSame(foo, foo2);
        Assertions.assertNotSame(foo1, foo2);
    }
}



# Mengubah Nama Bean
Jika kita ingin mengubah nama bean, kita bisa menggunakan method value() milik annotation @Bean

@Configuration
public class BeanNameConfiguration {
    @Primary
    @Bean(name = "fooFirst")
    public Foo foo1(){
        return new Foo();
    }   
    @Bean(name = "fooSecond")
    public Foo foo2(){
        return new Foo();
    }
}
public class BeanNameTest {   
    private ApplicationContext applicationContext;   
    @BeforeEach
    void setup(){
        applicationContext= new AnnotationConfigApplicationContext(BeanNameConfiguration.class);
    }   
    @Test
    void testBeanName(){
        Foo foo = applicationContext.getBean(Foo.class);
        Foo fooFirst = applicationContext.getBean("fooFirst",Foo.class);
        Foo fooSecond = applicationContext.getBean("fooSecond",Foo.class);
        
        Assertions.assertSame(foo, fooFirst);
        Assertions.assertNotSame(foo, fooSecond);
    }
}



# Dependency Injection
Dependency Injection (DI) adalah teknik dimana kita bisa mengotomatisasi proses pembuatan object yang tergantung dengan object lain, atau kita sebut dependencies Dependencies akan secara otomatis di-inject (dimasukkan) kedalam object yang membutuhkannya

@AllArgsConstructor
@Data
public class FooBar {
    
    private Foo foo;
    
    private Bar bar;
}



# Memilih Depedency
Saat terdapat duplicate bean dengan tipe data yang sama, secara otomatis Spring akan memilih bean yang primary Namun kita juga bisa memilih secara manual jika memang kita inginkan Kita bisa menggunakan annotation @Qualifier(value=”namaBean”) pada parameter di method nya

@Bean
public FooBar fooBar(@Qualifier("fooSecond") Foo foo, Bar bar){
    return new FooBar(foo, bar);
}
Foo foo = applicationContext.getBean("fooSecond", Foo.class);
Bar bar = applicationContext.getBean(Bar.class);
FooBar fooBar = applicationContext.getBean(FooBar.class);
        
Assertions.assertSame(foo, fooBar.getFoo());
Assertions.assertSame(bar, fooBar.getBar());



# Circular Dependencies
Circular dependencies adalah kasus dimana sebuah lingkaran dependency terjadi, misal bean A membutuhkan bean B, bean B membutuhkan bean C, dan ternyata bean C membutuhkan A Jika terjadi cyclic seperti ini, secara otomatis Spring bisa mendeteksinya, dan akan mengganggap bahwa itu adalah error

@Configuration
public class CyclicConfiguration {
    
    @Bean
    public CyclicA cyclicA(CyclicB cyclicB){
        return new CyclicA(cyclicB);
    }
    
    @Bean
    public CyclicB cyclicB(CyclicC cyclicC){
        return new CyclicB(cyclicC);
    }
    
    @Bean
    public CyclicC cyclicC(CyclicA cyclicA){
        return new CyclicC(cyclicA);
    }
}



# Depends On
Saat sebuah bean membutuhkan bean lain, secara otomatis bean tersebut akan dibuat setelah bean yang dibutuhkan dibuat Namun bagaimana jika bean tersebut tidak membutuhkan bean lain, namun kita ingin sebuah bean dibuat setelah bean lain dibuat? Jika ada kasus seperti itu, kita bisa menggunakan annotation @DependsOn(value={”namaBean”}) Secara otomatis, Spring akan memprioritaskan pembuatan bean yang terdapat di DependsOn terlebih dahulu

@Slf4j
@Configuration
public class DependsOnConfiguration {
    
    @Bean
    public Foo foo(){
        log.info("Create new Foo");
        return new Foo();
    }
    
    @Bean
    public Bar bar(){
        log.info("Create new Bar");
        return new Bar();
    }   
}
# Lazy Bean
Secara default, bean di Spring akan dibuat ketika aplikasi Spring pertama kali berjalan Oleh karena itu, kadang ketika aplikasi Spring pertama berjalan akan sedikit lambat, hal ini dikarenakan semua bean akan dibuat di awal Namun jika kita mau, kita juga bisa membuat sebuah bean menjadi lazy (malas), dimana bean tidak akan dibuat, sampai memang diakses atau dibutuhkan Untuk membuat sebuah bean menjadi lazy, kita bisa tambahkan annotation @Lazy pada bean tersebut

@Slf4j
@Configuration
public class DependsOnConfiguration {
    
    @Lazy
    @Bean
    @DependsOn({
        "bar"
    })
    public Foo foo(){
        log.info("Create new Foo");
        return new Foo();
    }
    
    @Bean
    public Bar bar(){
        log.info("Create new Bar");
        return new Bar();
    }
    
}



# Scope
Secara default strategy object di Spring adalah singleton, artinya hanya dibuat sekali, dan ketika kita akses, akan mengembalikan object yang sama Namun kita juga bisa mengubah scope bean yang kita mau di Spring Untuk mengubah scope sebuah bean, kita bisa tambahkan annotation @Scope(value=”namaScope”)

@Slf4j
@Configuration
public class ScopeConfiguration {
    
    @Bean
    @Scope("prototype")
    public Foo foo(){
        return new Foo();
    }
}



# Membuat Scope
Jika scope yang disediakan oleh Spring tidak bisa memenuhi kebutuhan kita, kita juga bisa membuat scope sendiri Caranya dengan membuat class yang implement interface Scope Selanjutnya untuk meregistrasikannya, kita bisa membuat bean CustomScopeConfigurer

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        counter ++;
        
        if(objects.size()==2){
            int index = (int)(counter % 2);
            return objects.get(index);
        } else {
            Object object = objectFactory.getObject();
            objects.add(object);
            return object;
        }
        
    }
    @Override
    public Object remove(String name) {
        if(!objects.isEmpty()){
            return objects.remove(0);
        }
        return null;
    }
Kode : Register Doubleton Scope
@Bean
    public CustomScopeConfigurer customScopeConfigurer(){
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("doubleton", new DoubletonScope());
        return configurer;
    }
    
    @Bean
    @Scope("doubleton")
    public Bar bar(){
        log.info("Create new Bar");
        return new Bar();
    }
Kode : Mengakses Doubleton Bean
    Bar bar1= applicationContext.getBean(Bar.class);
    Bar bar2= applicationContext.getBean(Bar.class);
    Bar bar3= applicationContext.getBean(Bar.class);
    Bar bar4= applicationContext.getBean(Bar.class);
        
     Assertions.assertSame(bar1, bar3);
     Assertions.assertSame(bar2, bar4);         
     Assertions.assertNotSame(bar1, bar2);
     Assertions.assertNotSame(bar3, bar4);



# Life Cycle Callback
Secara default, bean tidak bisa tahu alur hidup Spring ketika selesai membuat bean dan ketika akan menghancurkan bean Jika kita tertarik untuk bereaksi ketika alur hidup Spring terjadi, maka kita bisa implements interface InitializingBean dan DisposableBean InitializingBean digunakan jika kita ingin bereaksi ketika Spring selesai membuat bean

@Slf4j
public class Connection implements InitializingBean, DisposableBean{

    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("Connection is ready to be used");
    }

    @Override
    public void destroy() throws Exception {
        log.info("Connection is closed");
    } 
}
Kode : LifeCycle Configuration
@Configuration
public class LifeCycleConfiguration {
    
    @Bean
    public Connection connection(){
        return new Connection();
    }
}



# Life Cycle Annotation
Selain menggunakan interface InitializingBean dan DisposableBean, kita juga bisa menggunakan annotation untuk mendaftarkan callback method untuk lifecycle Pada annotation @Bean, terdapat method initMethod() dan destoyMethod() initMethod() digunakan untuk meregistrasikan method yang akan dipanggil ketika bean selesai dibuat destroyMethod() digunakan untuk meregistrasikan method yang akan dipanggil ketika bean akan dihancurkan

@Slf4j
public class Server {

    public void start(){
        log.info("Start Server");
    }

    public void stop(){
        log.info("Stop Server");
    }
}
@Bean (initMethod = "start", destroyMethod = "stop")
    public Server server(){
        return new Server();
    }



@PostConstruct dan @PreDestroy
@PostConstruct merupakan method yang ditandai harus dipanggil ketika bean selesai dibuat @PreDestroy merupakan method yang ditandai harus dipanggil ketika bean akan dihancurkan

@Slf4j
public class Server {
    
    @PostConstruct
    public void start(){
        log.info("Start Server");
    }
    
    @PreDestroy
    public void stop(){
        log.info("Stop Server");
    }
}



# Import
Biasanya kita akan membuat banyak sekali, tergantung seberapa kompleks aplikasi kita Spring mendukung import Configuration Class lain jika dibutuhkan Kita bisa menggunakan annotation @Import, lalu sebutkan Configuration Class mana yang ingin kita import Ketika kita melakukan import, kita bisa import lebih dari satu class

@Configuration
public class FooConfiguration {
    
    @Bean
    @Primary
    public Foo foo(){
        return new Foo();
    }
}
@Configuration
public class BarConfiguration {
    
    @Bean
    public Bar bar(){
        return new Bar();
    }
}
@Configuration
@Import({
    FooConfiguration.class,
    BarConfiguration.class
})
public class MainConfiguration {
    
}



# Component Scan
Spring memiliki fitur component scan, dimana kita bisa secara otomatis mengimport Configuration di sebuah package dan sub package nya secara otomatis Untuk melakukan itu, kita bisa gunakan annotation @ComponentScan

@Configuration
@ComponentScan(basePackages = {
    "com.fariz.farizbelajarspringdasar.configuration"
})
public class ScanConfiguration {
    
}



# Multiple Constructor
Seperti di awal disebutkan bahwa Spring hanya mendukung satu constructor untuk Dependency Injection nya Namun bagaimana jika terdapat multiple constructor? Jika pada kasus seperti ini, kita harus menandai constructor mana yang akan digunakan oleh Spring Caranya kita bisa menggunakan annotation @Autowired

@Component
public class ProductService {
    
    @Getter
    private ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository){
        this.productRepository = productRepository;
    }
    
    public ProductService(ProductRepository productRepository, String name){
        this.productRepository = productRepository;
    }
}



# Setter-based Dependency Injection
Selain menggunakan constructor parameter, kita juga bisa menggunakan setter method jika ingin melakukan dependency injection Namun khusus untuk setter method, kita perlu menambah annotation @Autowired pada setter method nya Secara otomatis Spring akan mencari bean yang dibutuhkan di setter method yang memiliki annotation @Autowired Setter-based DI juga bisa digabung dengan Constructor-based DI

@Component
public class CategoryService {
    
    @Getter
    private CategoryRepository categoryRepository;
    
    @Autowired
    public void setCategoryRepository(CategoryRepository categoryRepository){
        this.categoryRepository = categoryRepository;
    }
}



# Field-based Dependency Injection
Selain constructor dan setter, kita juga bisa melakukan dependency injection langsung menggunakan field Caranya sama dengan setter, kita bisa tambahkan @Autowired pada fieldnya Secara otomatis Spring akan mencari bean dengan tipe data tersebut Field-based DI bisa digabung sekaligus dengan Setter-based DI dan Constructor-based DI Khusus Field-based DI, Spring sendiri sudah tidak merekomendasikan penggunaan cara melakukan DI dengan Field

@Component
public class CustomerService {
    
    @Getter
    @Autowired
    private CustomerRepository CustomerRepository;
}



# Qualifier
Seperti yang sudah dijelaskan di awal, jika terdapat bean dengan tipe data yang sama lebih dari satu, maka secara otomatis Spring akan bingung memilih bean yang mana yang akan digunakan Kita perlu memilih salah satu menjadi primary, yang secara otomatis akan dipilih oleh Spring Namun jika kita ingin memilih bean secara manual, kita juga bisa menggunakan @Qualifier Kita bisa tambahkan @Qualifier di constructor parameter, di setter method atau di field

@Component
public class CustomerService {
    
    @Getter
    @Autowired
    @Qualifier("normalCustomerRepository")
    private CustomerRepository normalCustomerRepository;
    
    @Getter
    @Autowired
    @Qualifier("premiumCustomerRepository")
    private CustomerRepository premiumCustomerRepository;
}



# Optional Dependency
Secara default, semua dependency itu wajib Artinya jika Spring tidak bisa menemukan bean yang dibutuhkan pada saat DI, maka secara otomatis akan terjadi error Namun jika kita memang ingin membuat sebuah dependency menjadi Optional, artinya tidak wajib

@Configuration
public class OptionalConfiguration {
    
    @Bean
    public Foo foo(){
        return new Foo();       
    }
    
    @Bean
    public FooBar fooBar(Optional<Foo> foo, Optional<Bar> bar){
        return new FooBar(foo.orElse(null), bar.orElse(null));
    }
}



# Factory Bean
Kadang ada kasus dimana sebuah class misal bukanlah milik kita, misal class third party library Sehingga agak sulit jika kita harus menambahkan annotation pada class tersebut Pada kasus seperti ini, cara terbaik untuk membuat bean nya adalah dengan menggunakan @Bean method Atau di Spring, kita juga bisa menggunakan @Component, namun kita perlu wrap dalam class Factory Bean

@Data
public class PaymentGatewayClient {
    
    private String endpoint;
    
    private String publicKey;
    
    private String privateKey;
}


Kode : Factory Bean

@Component("paymentGatewayClient")
public class PaymentGatewayClientFactoryBean implements FactoryBean<PaymentGatewayClient>{

    @Override
    public PaymentGatewayClient getObject() throws Exception {
        PaymentGatewayClient client = new PaymentGatewayClient();
        client.setEndpoint("https://example.com");
        client.setPrivateKey("private");
        client.setPublicKey("public");
        return client;
    }
}


Kode : Configuration

@Configuration
@Import({
    PaymentGatewayClientFactoryBean.class
})
public class FactoryConfiguration {
    
}


Kode : Mengakses Bean

PaymentGatewayClient client = applicationContext.getBean(PaymentGatewayClient.class);
        
Assertions.assertEquals("https://example.com", client.getEndpoint());
Assertions.assertEquals("private", client.getPrivateKey());
Assertions.assertEquals("public", client.getPublicKey());



# Inheritance
Saat kita mengakses bean, kita bisa langsung menyebutkan tipe class bean tersebut, atau bisa juga dengan parent class / parent interface bean Misal jika kita memiliki sebuah interface bernama MerchantService, lalu kita memiliki bean dengan object implementasi class nya MerchantServiceImpl, maka untuk mengakses bean nya, kita tidak hanya bisa menggunakan tipe MerchantServiceImpl, namun juga bisa dengan MerchantService Namun perlu berhati-hati, jika misal MerchantService memiliki banyak bean turunan, pastikan tidak terjadi error duplicate

public interface MerchantService {
    
}
@Component
public class MerchantServiceImpl implements MerchantService{
    
}
@Configuration
@Import(MerchantServiceImpl.class)
public class InheritanceConfiguration {
    
}



# Bean Factory
ApplicationContext adalah interface turunan dari BeanFactory BeanFactory merupakan kontrak untuk management bean di Spring Method-method yang sebelumnya kita gunakan untuk mengambil bean, sebenarnya merupakan method kontrak dari interface BeanFactory


Listable Bean Factory
Listable Bean Factory adalah turunan dari Bean Factory yang bisa kita gunakan untuk mengakses beberapa bean sekaligus Dalam beberapa kasus, ini sangat berguna, seperti misal kita ingin mengambil semua bean dengan tipe tertentu

ObjectProvider<Foo> fooObjectProvider = applicationContext.getBeanProvider(Foo.class);

Map<String, Foo> beans = applicationContext.getBeansOfType(Foo.class);



# Bean Post Processor
Bean Post Processor merupakan sebuah interface yang bisa kita gunakan untuk memodifikasi proses pembuatan bean di Application Context Bean Post Processor mirip seperti middleware, yang diakses sebelum bean di initialized dan setelah bean di initialized

public interface IdAware {
    
    void setId(String Id);
}
Kode : Bean Post Processor

@Component
public class IdGeneratorBeanPostProcessor implements BeanPostProcessor {

@Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException{
        if(bean instanceof IdAware){
            IdAware idAware = (IdAware) bean;
            idAware.setId(UUID.randomUUID().toString());
    }
        return bean;
    }
Component

@Component
public class Car implements IdAware{
    
    @Getter
    private String id;

    @Override
    public void setId(String id) {
        this.id = id;
    }
}



# Ordered
Saat kita membuat Bean Post Processor, kita bisa membuat lebih dari satu Kadang ada kasus saat membuat beberapa Bean Post Processor, kita ingin membuat yang berurutan Sayangnya secara default, Spring tidak menjamin urutan eksekusi nya Agar kita bisa menentukan urutannya, kita bisa menggunakan interface Ordered

public interface IdAware {
    
    void setId(String Id);
    
    String getId();
}
Kode : Id Generator

@Component
public class IdGeneratorBeanPostProcessor implements BeanPostProcessor, Ordered{
    
    @Override
    public int getOrder() {
        return 1;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException{
        log.info("Id Generator Processor for Bean {}", beanName);
        if(bean instanceof IdAware){
            log.info("Set Id Generator for Bean {}", beanName);
            IdAware idAware = (IdAware) bean;
            idAware.setId(UUID.randomUUID().toString());
    }
        return bean;
    }   
}
Kode : Prefix Id Generator

@Component
public class PrefixIdGeneratorBeanPostProcessor implements BeanPostProcessor, Ordered{
    
    @Override
    public int getOrder() {
        return 2;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException{
        log.info("Prefix Id Generator Processor for Bean {}", beanName);
        if(bean instanceof IdAware){
            log.info("Prefix Set Id Generator for Bean {}", beanName);
            IdAware idAware = (IdAware) bean;
            idAware.setId("FARIZ-" +idAware.getId());
    }
        return bean;
    }
}



# Aware
Aware adalah super interface yang digunakan untuk semua Aware interface Aware ini diperuntukkan untuk penanda agar Spring melakukan injection object yang kita butuhkan Mirip seperti yang sudah kita lakukan ketika membuat IdAware menggunakan IdGenerator Bean Post Processor

@Component
public class AuthService implements ApplicationContextAware, BeanNameAware{
    
    @Getter
    private String beanName;
    
    @Getter
    private ApplicationContext applicationContext;
}



# Bean Factory Post Processor
Secara default, mungkin kita tidak akan pernah sama sekali membuat Application Context secara manual Namun kadang ada keadaan kita ingin memodifikasi secara internal Application Context Spring merekomendasikan kita untuk membuat Bean Factory Post Processor

@Component
public class FooBeanFactoryPostProcessor implements BeanDefinitionRegistryPostProcessor{

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        GenericBeanDefinition definition = new GenericBeanDefinition();
        definition.setScope("singleton");
        definition.setBeanClass(Foo.class);
        
        registry.registerBeanDefinition("foo", definition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory clbf) throws BeansException {

    }    
}



# Event Listener
Spring memiliki fitur Event Listener yang bisa kita gunakan untuk komunikasi antar class menggunakan Event Event di Spring merupakan object turunan dari ApplicationEvent, sedangkan Listener di Spring merupakan turunan dari ApplicationListener


Application Event Publisher
Ketika kita ingin mengirimkan event ke listener, kita bisa menggunakan object ApplicationEventPublisher, dimana ApplicationEventPublisher juga merupakan super interface dari ApplicationContext Atau kita bisa menggunakan ApplicationEventPublisherAware untuk mendapatkan object ApplicationEventPublisher

public class LoginSuccessEvent extends ApplicationEvent{
    
    @Getter
    private final User user;
    
    public LoginSuccessEvent(User user){
        super(user);
        this.user = user;
    }
    
}
Kode : Listener

@Component
@Slf4j
public class LoginSuccessListener implements ApplicationListener<LoginSuccessEvent>{

    @Override
    public void onApplicationEvent(LoginSuccessEvent event) {
        log.info("Success login for user {}", event.getUser());
    }
    
}
Kode : Mengirim Event

@Component
public class UserService implements ApplicationEventPublisherAware{

    private ApplicationEventPublisher applicationEventPublisher;
    
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
    
    public boolean login(String username, String password){
        if(isLoginSuccess(username, password)){
            applicationEventPublisher.publishEvent(new LoginSuccessEvent(new User(username)));
            return true;
        }else{
            return false;
        }
    }
    
    public boolean isLoginSuccess(String username, String password){
        return "fariz".equals(username) && "fariz".equals(password);
    }
    
}
Mengakses Bean

public class EventListenerTest {
    
    @Configuration
    @Import({
        UserService.class,
        LoginSuccessListener.class,
        LoginAgainSuccessListener.class,
        UserListener.class
    })
    public static class TestConfiguration{
        
    }
    
    private ConfigurableApplicationContext applicationContext;
    
    @BeforeEach
    void setUp(){
        applicationContext = new AnnotationConfigApplicationContext(TestConfiguration.class);
        applicationContext.registerShutdownHook();
    }
    
    @Test
    void testEvent(){
        
        UserService userService = applicationContext.getBean(UserService.class);
        userService.login("fariz", "fariz");
        userService.login("fariz", "salah");
        userService.login("kurniawan", "salah");
    }
}



# Event Listener Annotation
Selain menggunakan interface ApplicationListener, kita juga bisa menggunakan Annotation untuk membuat Listener Ini lebih flexible dibanding menggunakan interface ApplicationListener Kita bisa menggunakan annotation @EventListener

@Slf4j
@Component
public class UserListener {
    
    @EventListener(classes = LoginSuccessEvent.class)
    public void onLoginSuccessEvent(LoginSuccessEvent event){
        log.info("Success login again for user {}", event.getUser());
    }
}
Cara Kerja Event Listener Annotation? Annotation @EventListener bekerja dengan menggunakan Bean Post Processor bernama EventListenerMethodProcessor EventListenerMethodProcessor mendeteksi jika ada bean yang memiliki method dengan annotation @EventListener, maka secara otomatis akan membuat listener baru, lalu meregistrasikannya ke ApplicationContext.addApplicationListener(listener)


# Spring Application
Selain @SpringBootApplication, untuk membuat Application Context nya, kita tidak perlu membuat manual, kita bisa gunakan class SpringApplication Secara otomatis SpringApplication akan membuat ApplicationContext dan melakukan hal-hal yang dibutuhkan secara otomatis

@SpringBootApplication
public class FooApplication {
    
    @Bean
    public Foo foo(){
        return new Foo();
    }
}
Spring Boot Test Untuk membuat unit test di Spring Boot, kita bisa menggunakan annotation @SpringBootTest(classes={YourConfiguration.class}) Selanjutnya kita tidak perlu mengambil bean secara manual lagi menggunakan ApplicationContext, kita bisa menggunakan DI secara langsung di unit test nya menggunakan @Autowired

@SpringBootTest(classes = FooApplication.class)
public class FooApplicationTest {
    
    @Autowired
    Foo foo;
    
    @Test
    void testSpringBoot(){
        Assertions.assertNotNull(foo);
    }
}



# Startup Failure
FailureAnalyzer digunakan untuk melakukan analisa ketika terjadi error startup yang menyebabkan aplikasi tidak mau berjalan


# Banner
Spring Boot memiliki fitur banner, dimana saat aplikasi Spring Boot berjalan, kita bisa menampilkan tulisan banner di console Secara default fitur banner ini akan menyala dan akan mencari tulisan banner di classpath dengan nama banner.txt Jika tidak ada file tersebut, maka secara otomatis akan menampilkan tulisan banner Spring Boot Salah satu contoh tempat untuk membuat banner adalah http://www.bagill.com/ascii-sig.php


# Customizing Spring Application
Kadang ada kalanya kita ingin melakukan pengaturan di Spring Application sebelum Application Context nya dibuat https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html Kita bisa menggunakan langsung SpringApplication, atau bisa juga menggunakan SpringApplicationBuilder https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/builder/SpringApplicationBuilder.html


# Spring Application Event
Sebelumnya kita sudah belajar tentang Event Listener Di Spring Boot, terdapat banyak sekali Event yang dikirim ketika aplikasi Spring Boot berjalan Jika kita ingin, kita bisa membuat Listener untuk menerima event tersebut


Menambah Listener Beberapa Event di Spring Boot Application Event di trigger bahkan sebelum Spring membuat Application Context Oleh karena itu, jika kita buat menggunakan bean, bisa saja beberapa listener tidak akan dipanggil, karena bean nya belum dibuat Agar lebih aman, kita bisa menambahkan listener ketika membuat SpringApplication

@Slf4j
public class AppStartingListener implements ApplicationListener<ApplicationStartedEvent>{

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        log.info("Application starting");
    }
    
}



# Command Line Runner
Saat kita membuat aplikasi, kadang kita butuh argument yang diberikan pada main method Spring Application bisa mengirim data argument tersebut secara otomatis ke bean yang kita buat Kita hanya butuh membuat bean dari CommandLineRunner CommandLineRunner secara otomatis akan di jalankan ketika Spring Application berjalan


# Application Runner
Selain CommandLineRunner, Spring Boot menyediakan fitur ApplicationRunner Penggunaan ApplicationRunner sama seperti CommandLineRunnnner, hanya saja argument nya sudah di wrap dalam object ApplicationArguments Yang menarik dari ApplicationArguments adalah, memiliki fitur parsing untuk command line argument

@Slf4j
@Component
public class SimpleRunner implements ApplicationRunner{

    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> profiles = args.getOptionValues("profiles");
        log.info("Profiles : {}", profiles);
    }
    
}



# Spring Boot Plugin
Saat kita membuat project Spring Boot, secara otomatis terdapat spring-boot-plugin di project maven kita Plugin ini bisa digunakan untuk mempermudah saat kita menjalankan aplikasi Spring kita Kita bisa gunakan perintah : mvn spring-boot:run Untuk menjalankan aplikasi Spring Boot kita, kita harus memastikan bahwa hanya ada satu main class


# Distribution File
Spring Boot plugin juga sudah menyediakan cara membuat distribution file aplikasi kita Plugin ini akan mendeteksi main class di project kita, lalu membundle aplikasi kita beserta dependency yang dibutuhkan dalam satu file jar Pastikan hanya terdapat satu main class, karena jika lebih dari satu, maka spring boot plugin akan melakukan komplen Kita cukup gunakan perintah : mvn package Secara otomatis akan terbuat single jar application
