# Testing Your Templates and Views

## Tutorial-1

### Template

```java
import id.ac.ui.cs.advprog.tutorial1.observer.core.Guild;
import id.ac.ui.cs.advprog.tutorial1.observer.core.Quest;
import id.ac.ui.cs.advprog.tutorial1.observer.service.GuildServiceImpl;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(controllers = ObserverController.class)
public class ObserverControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private Guild guild;

    @MockBean
    private GuildServiceImpl guildService;
	
    // Put your tests here with the decorator @Test
}
```

### Get Request

See that `perform` will request your page with a method described in the parameter inside, then there is a `and

```java
@Test
public void whenCreateQuestURLIsAccessedItShouldContainCorrectQuestModel() throws Exception {
    mockMvc.perform(get("/create-quest"))
        .andExpect(status().isOk())
        .andExpect(model().attributeExists("quest"))
        .andExpect(view().name("observer/questForm"));
}
```

### Post Request

```java
@Test
public void whenAddQuestURLIsAccessedItShouldCallGuildServiceAddQuest() throws Exception {
    Quest newQuest = new Quest();
    newQuest.setTitle("Dummy");
    newQuest.setType("R");

    mockMvc.perform(post("/add-quest")
        .flashAttr("quest", newQuest))
        .andExpect(handler().methodName("addQuest"))
        .andExpect(status().is3xxRedirection())
        .andExpect(redirectedUrl("/adventurer-list"));
}
```

### Extra Notes

Perhatikan bahwa `assertEquals()` dan `assertSame()` itu mirip perbandingannya Javascript, `assertSame()` itu bakal ngassert kalau objectnya sama, sementara assertEquals itu cuma nilai `.equaTo()` nya

Perhatikan juga biasanya test itu ada `@BeforeEach` yang bakal dijalanin setiap kali masuk ke suatu method, ada juga `@Before` yang bakal dijalanin sekali aja dalam satu class.

## Tutorial-3 Adapter

### Controller

```java
@WebMvcTest(controllers = AdapterController.class)
public class AdapterControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private WeaponServiceImpl weaponService;

    @Test
    public void whenBattleHomeURLIsAccessedItShouldContainWeaponChoiceAndBattleLog() throws Exception {
        mockMvc.perform(get("/battle"))
                .andExpect(status().isOk())
                .andExpect(handler().methodName("battleHome"))
                .andExpect(model().attributeExists("weapons"))
                .andExpect(model().attributeExists("logs"))
                .andExpect(view().name("adapter/home"));
        verify(weaponService, times(1)).findAll();
        verify(weaponService, times(1)).getAllLogs();
    }

    @Test
    public void whenAttackURLIsAccessedItShouldCallAttackWithWeapon() throws Exception {
        mockMvc.perform(post("/battle/attack")
                .param("weaponName", "Ionic Bow")
                .param("attackType", "1"))
                .andExpect(handler().methodName("attackWithWeapon"))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/battle"));
        verify(weaponService, times(1)).attackWithWeapon("Ionic Bow", 1);
    }
}
```

### Bow Core Interface

```java
public class BowTest {
    private Class<?> bowClass;

    @BeforeEach
    public void setup() throws Exception {
        bowClass = Class.forName("id.ac.ui.cs.advprog.tutorial3.adapter.core.bow.Bow");
    }

    @Test
    public void testBowIsAPublicInterface() {
        int classModifiers = bowClass.getModifiers();

        assertTrue(Modifier.isPublic(classModifiers));
        assertTrue(Modifier.isInterface(classModifiers));
    }

    @Test
    public void testBowHasShootArrowAbstractMethod() throws Exception {
        Class<?>[] shootArrowArgs = new Class[1];
        shootArrowArgs[0] = boolean.class;
        Method shootArrow = bowClass.getDeclaredMethod("shootArrow", shootArrowArgs);
        int methodModifiers = shootArrow.getModifiers();

        assertTrue(Modifier.isPublic(methodModifiers));
        assertTrue(Modifier.isAbstract(methodModifiers));
        assertEquals(1, shootArrow.getParameterCount());
    }

    @Test
    public void testBowHasGetNameAbstractMethod() throws Exception {
        Method getName = bowClass.getDeclaredMethod("getName");
        int methodModifiers = getName.getModifiers();

        assertTrue(Modifier.isPublic(methodModifiers));
        assertTrue(Modifier.isAbstract(methodModifiers));
        assertEquals(0, getName.getParameterCount());
    }

    @Test
    public void testBowHasGetHolderNameAbstractMethod() throws Exception {
        Method getHolderName = bowClass.getDeclaredMethod("getHolderName");
        int methodModifiers = getHolderName.getModifiers();

        assertTrue(Modifier.isPublic(methodModifiers));
        assertTrue(Modifier.isAbstract(methodModifiers));
        assertEquals(0, getHolderName.getParameterCount());
    }
}
```

### Bow Concrete Implementation Tests

```java
public class IonicBowTest {
    private Class<?> ionicBowClass;
    private IonicBow bow = new IonicBow("Venti");

    @BeforeEach
    public void setUp() throws Exception {
        ionicBowClass = Class.forName("id.ac.ui.cs.advprog.tutorial3.adapter.core.bow.IonicBow");
    }

    @Test
    public void testIonicBowIsConcreteClass() {
        assertFalse(Modifier.
                isAbstract(ionicBowClass.getModifiers()));
    }

    @Test
    public void testIonicBowIsABow() {
        Collection<Type> interfaces = Arrays.asList(ionicBowClass.getInterfaces());

        assertTrue(interfaces.stream()
                .anyMatch(type -> type.getTypeName()
                        .equals("id.ac.ui.cs.advprog.tutorial3.adapter.core.bow.Bow")));
    }

    @Test
    public void testIonicBowOverrideShootArrowMethod() throws Exception {
        Class<?>[] shootArrowArgs = new Class[1];
        shootArrowArgs[0] = boolean.class;
        Method shootArrow = ionicBowClass.getDeclaredMethod("shootArrow", shootArrowArgs);

        assertEquals("java.lang.String",
                shootArrow.getGenericReturnType().getTypeName());
        assertEquals(1,
                shootArrow.getParameterCount());
        assertTrue(Modifier.isPublic(shootArrow.getModifiers()));
    }

    @Test
    public void testIonicBowOverrideGetNameMethod() throws Exception {
        Method getName = ionicBowClass.getDeclaredMethod("getName");

        assertEquals("java.lang.String",
                getName.getGenericReturnType().getTypeName());
        assertEquals(0,
                getName.getParameterCount());
        assertTrue(Modifier.isPublic(getName.getModifiers()));
    }

    @Test
    public void testIonicBowOverrideGetHolderMethod() throws Exception {
        Method getHolderName = ionicBowClass.getDeclaredMethod("getHolderName");

        assertEquals("java.lang.String",
                getHolderName.getGenericReturnType().getTypeName());
        assertEquals(0,
                getHolderName.getParameterCount());
        assertTrue(Modifier.isPublic(getHolderName.getModifiers()));
    }

    @Test
    public void testIonicBowOutputTrueDescription() throws Exception {
        assertEquals("Venti", bow.getHolderName());
        assertEquals("Ionic Bow", bow.getName());
    }
    @Test
    public void testIonicBowOutputTrueAttackDescription() throws Exception {
        assertEquals("Separated one atom from the enemy", bow.shootArrow(false));
        assertEquals("Arrow reacted with the enemy's protons", bow.shootArrow(true));
    }
}
```

### Bow Adapter Tests

```java
public class BowAdapterTest {
    private Class<?> bowAdapterClass;
    private Class<?> bowClass;

    private Weapon bowAdapter;
    private Bow bow;

    @BeforeEach
    public void setUp() throws Exception {
        bowAdapterClass = Class.forName("id.ac.ui.cs.advprog.tutorial3.adapter.core.weaponadapters.BowAdapter");
        bowClass = Class.forName("id.ac.ui.cs.advprog.tutorial3.adapter.core.bow.Bow");
        bow = new IonicBow("Venti");
        bowAdapter = new BowAdapter(bow);
    }

    @Test
    public void testBowAdapterIsConcreteClass() {
        assertFalse(Modifier.
                isAbstract(bowAdapterClass.getModifiers()));
    }

    @Test
    public void testBowAdapterIsAWeapon() {
        Collection<Type> interfaces = Arrays.asList(bowAdapterClass.getInterfaces());

        assertTrue(interfaces.stream()
                .anyMatch(type -> type.getTypeName()
                        .equals("id.ac.ui.cs.advprog.tutorial3.adapter.core.weapon.Weapon")));
    }

    @Test
    public void testBowAdapterConstructorReceivesBowAsParameter() {
        Class<?>[] classArg = new Class[1];
        classArg[0] = bowClass;
        Collection<Constructor<?>> constructors = Arrays.asList(
                bowAdapterClass.getDeclaredConstructors());

        assertTrue(constructors.stream()
                .anyMatch(type -> Arrays.equals(
                        type.getParameterTypes(), classArg)));
    }

    @Test
    public void testBowAdapterOverrideNormalAttackMethod() throws Exception {
        Method normalAttack = bowAdapterClass.getDeclaredMethod("normalAttack");

        assertEquals("java.lang.String",
                normalAttack.getGenericReturnType().getTypeName());
        assertEquals(0,
                normalAttack.getParameterCount());
        assertTrue(Modifier.isPublic(normalAttack.getModifiers()));
    }

    @Test
    public void testBowAdapterOverrideChargedAttackMethod() throws Exception {
        Method chargedAttack = bowAdapterClass.getDeclaredMethod("chargedAttack");

        assertEquals("java.lang.String",
                chargedAttack.getGenericReturnType().getTypeName());
        assertEquals(0,
                chargedAttack.getParameterCount());
        assertTrue(Modifier.isPublic(chargedAttack.getModifiers()));
    }

    @Test
    public void testBowAdapterOverrideGetNameMethod() throws Exception {
        Method getName = bowAdapterClass.getDeclaredMethod("getName");

        assertEquals("java.lang.String",
                getName.getGenericReturnType().getTypeName());
        assertEquals(0,
                getName.getParameterCount());
        assertTrue(Modifier.isPublic(getName.getModifiers()));
    }

    @Test
    public void testBowAdapterOverrideGetHolderMethod() throws Exception {
        Method getHolderName = bowAdapterClass.getDeclaredMethod("getHolderName");

        assertEquals("java.lang.String",
                getHolderName.getGenericReturnType().getTypeName());
        assertEquals(0,
                getHolderName.getParameterCount());
        assertTrue(Modifier.isPublic(getHolderName.getModifiers()));
    }

    @Test
    public void testBowAdapterSameBehaviorName() throws Exception {
        assertEquals("Venti", bowAdapter.getHolderName());
        assertEquals("Ionic Bow", bowAdapter.getName());
    }

    @Test
    public void testBowAdapterSameBehaviorAttack() throws Exception {
        assertEquals(bow.shootArrow(false), bowAdapter.normalAttack());

        assertEquals("Entered aim shot mode", bowAdapter.chargedAttack());

        assertEquals(bow.shootArrow(true), bowAdapter.normalAttack());
        assertEquals("Arrow reacted with the enemy's protons", bowAdapter.normalAttack());

        assertEquals("Leaving aim shot mode", bowAdapter.chargedAttack());

        assertEquals(bow.shootArrow(false), bowAdapter.normalAttack());
        assertEquals("Separated one atom from the enemy", bowAdapter.normalAttack());
    }
}
```

### Repository Tests

```java
@Repository
public class SpellbookRepositoryTest {
    private SpellbookRepository spellbookRepository;

    @Mock
    private Map<String, Spellbook> spellbooks;

    private Spellbook sampleSpellbook;

    @BeforeEach
    public void setUp() {
        spellbookRepository = new SpellbookRepositoryImpl();
        spellbooks = new HashMap<>();
        sampleSpellbook = new Heatbearer("Kocheng");
        spellbooks.put(sampleSpellbook.getName(), sampleSpellbook);
    }

    @Test
    public void whenSpellbookRepoFindAllItShouldReturnSpellbookList() {
        ReflectionTestUtils.setField(spellbookRepository, "spellbooks", spellbooks);
        List<Spellbook> acquiredSpellbooks = spellbookRepository.findAll();

        assertThat(acquiredSpellbooks).isEqualTo(new ArrayList<>(spellbooks.values()));
    }

    @Test
    public void whenSpellbookRepoFindByAliasItShouldReturnSpellbookList() {
        ReflectionTestUtils.setField(spellbookRepository, "spellbooks", spellbooks);
        Spellbook acquiredSpellbook = spellbookRepository.findByAlias(sampleSpellbook.getName());

        assertThat(acquiredSpellbook).isEqualTo(sampleSpellbook);
    }

    @Test
    public void whenSpellbookRepoSaveItShouldSaveSpellbook() {
        ReflectionTestUtils.setField(spellbookRepository, "spellbooks", spellbooks);
        Spellbook newSpellbook = new TheWindjedi("Mei Mei");
        spellbookRepository.save(newSpellbook);
        Spellbook acquiredSpellbook = spellbookRepository.findByAlias(newSpellbook.getName());

        assertThat(acquiredSpellbook).isEqualTo(newSpellbook);
    }
}
```

### Service Tests

```java
@ExtendWith(MockitoExtension.class)
public class WeaponServiceTest {
    private Class<?> weaponServiceClass;

    @Mock
    BowRepository bowRepository;

    @Mock
    SpellbookRepository spellbookRepository;

    @Spy
    WeaponRepositoryImpl weaponRepository = new WeaponRepositoryImpl();

    @Mock
    LogRepository logRepository;

    @InjectMocks
    WeaponService weaponService = new WeaponServiceImpl();

    @BeforeEach
    public void setup() throws Exception {
        weaponServiceClass = Class.forName(
                "id.ac.ui.cs.advprog.tutorial3.adapter.service.WeaponServiceImpl");
    }

    @Test
    public void testWeaponServiceHasFindAllMethod() throws Exception {
        Method findAll = weaponServiceClass.getDeclaredMethod("findAll");
        int methodModifiers = findAll.getModifiers();
        assertTrue(Modifier.isPublic(methodModifiers));

        ParameterizedType pt = (ParameterizedType) findAll.getGenericReturnType();
        assertEquals(List.class, pt.getRawType());
        assertTrue(Arrays.asList(pt.getActualTypeArguments()).contains(Weapon.class));
    }

    @Test
    public void testWeaponServiceFindAllReturnCorrectWeaponAmount() {
        List<Bow> mockListBow = new ArrayList<>();
        mockListBow.add(new UranosBow("Ganyu"));
        mockListBow.add(new IonicBow("Faisal"));

        List<Spellbook> mockListSpellbook = new ArrayList<>();
        mockListSpellbook.add(new TheWindjedi("Klee"));

        weaponRepository.save(new StaffOfHoumo("Who Tao"));

        when(bowRepository.findAll()).thenReturn(mockListBow);
        when(spellbookRepository.findAll()).thenReturn(mockListSpellbook);

        assertEquals(4, weaponService.findAll().size());
        verify(weaponRepository, atLeastOnce()).findAll();
        verify(bowRepository, atLeastOnce()).findAll();
        verify(spellbookRepository, atLeastOnce()).findAll();
    }

    @Test
    public void testWeaponServiceHasGetAllLogsMethod() throws Exception {
        Method getAllLogs = weaponServiceClass.getDeclaredMethod("getAllLogs");
        int methodModifiers = getAllLogs.getModifiers();
        assertTrue(Modifier.isPublic(methodModifiers));

        ParameterizedType pt = (ParameterizedType) getAllLogs.getGenericReturnType();
        assertEquals(List.class, pt.getRawType());
        assertTrue(Arrays.asList(pt.getActualTypeArguments()).contains(String.class));
    }

    @Test
    public void testWeaponServiceGetAllLogsCorrectlyImplemented() {
        weaponService.getAllLogs();
        verify(logRepository, atLeastOnce()).findAll();
    }


    @Test
    public void testWeaponServiceHasAttackWithWeaponMethod() throws Exception {
        Method attackWithWeapon = weaponServiceClass.getDeclaredMethod("attackWithWeapon", String.class, int.class);
        int methodModifiers = attackWithWeapon.getModifiers();
        assertTrue(Modifier.isPublic(methodModifiers));
    }

    @Test
    public void testWeaponServiceAttackWithWeaponCorrectlyImplemented() {
        Weapon mockWeapon = new SeawardPride("Usep");
        when(weaponRepository.findByAlias(any(String.class))).thenReturn(mockWeapon);
        weaponService.attackWithWeapon("Seaward Pride", 1);
        verify(weaponRepository, atLeastOnce()).findByAlias(any(String.class));
        verify(logRepository, atLeastOnce()).addLog(any(String.class));
        verify(weaponRepository, atLeastOnce()).save(mockWeapon);
    }

    @Test
    public void testABowNotYetMadeABowAdapterAndSavedInWeaponRepositoryButFindByAliasOnBowFound() {
        Bow ventiBow = new IonicBow("Venti");
        when(weaponRepository.findByAlias(any(String.class))).thenReturn(null);
        when(bowRepository.findByAlias(any(String.class))).thenReturn(ventiBow);
        weaponService.attackWithWeapon("Ionic Bow", 0);
        verify(weaponRepository, atLeastOnce()).findByAlias(any(String.class));
        verify(bowRepository, atLeastOnce()).findByAlias(any(String.class));
    }
    @Test
    public void testASpellbookNotYetMadeASpellbookAdapterAndSavedInWeaponRepositoryButFindByAliasOnSpellbookFound() {
        Spellbook lisaSpellbook = new Heatbearer("Lisa");
        when(weaponRepository.findByAlias(any(String.class))).thenReturn(null);
        when(bowRepository.findByAlias(any(String.class))).thenReturn(null);
        when(spellbookRepository.findByAlias(any(String.class))).thenReturn(lisaSpellbook);
        weaponService.attackWithWeapon("Heat Bearer", 0);
        verify(weaponRepository, atLeastOnce()).findByAlias(any(String.class));
        verify(bowRepository, atLeastOnce()).findByAlias(any(String.class));
        verify(spellbookRepository, atLeastOnce()).findByAlias(any(String.class));
    }

    @Test
    public void testFindAllButAlreadyFoundABowAdapterOnWeaponRepository() {
        Bow ventiBow = new IonicBow("Venti");
        BowAdapter ventiAdapter = new BowAdapter(ventiBow);
        List <Bow> mockListBow = new ArrayList<>();
        mockListBow.add(ventiBow);
        when(bowRepository.findAll()).thenReturn(mockListBow);
        when(weaponRepository.findByAlias(any(String.class))).thenReturn(ventiAdapter);
        weaponService.findAll();
        verify(bowRepository, atLeastOnce()).findAll();
        verify(weaponRepository, atLeastOnce()).findByAlias(any(String.class));
    }

    @Test
    public void testFindAllButAlreadyFoundASpellbookAdapterOnWeaponRepository() {
        Spellbook lisaSpellbook = new Heatbearer("Lisa");
        SpellbookAdapter lisaAdapter = new SpellbookAdapter(lisaSpellbook);
        List <Spellbook> mockListSpellbook = new ArrayList<>();
        mockListSpellbook.add(lisaSpellbook);
        when(spellbookRepository.findAll()).thenReturn(mockListSpellbook);
        when(weaponRepository.findByAlias(any(String.class))).thenReturn(lisaAdapter);
        weaponService.findAll();
        verify(spellbookRepository, atLeastOnce()).findAll();
        verify(weaponRepository, atLeastOnce()).findByAlias(any(String.class));
    }
}
```

## Tutorial-3 Facade

### Singleton Tests

```java
@ExtendWith(MockitoExtension.class)
public class CodexTranslatorTest {

    @Spy
    AlphaCodex oldCodex;
    @Spy
    RunicCodex newCodex;

    @Test
    public void testCodexHasTranslateStaticMethod() throws Exception {
        Class<?> translatorClass = Class.forName(
                "id.ac.ui.cs.advprog.tutorial3.facade.core.misc.CodexTranslator");
        Method translate = translatorClass.getDeclaredMethod("translate", Spell.class, Codex.class);
        int methodModifiers = translate.getModifiers();
        assertTrue(Modifier.isPublic(methodModifiers));
        assertTrue(Modifier.isStatic(methodModifiers));
    }

    @Test
    public void testCodexTranslateAlphaToRunicProperly(){
        String text = "Safira and I went to a blacksmith to forge our sword";
        Codex codex = AlphaCodex.getInstance();
        Codex targetCodex = RunicCodex.getInstance();
        Spell spell = new Spell(text, codex);
        String expected = "eJcnBJ_JZz_._DxZM_MX_J_KsJLaNdnMb_MX_cXBvx_XAB_NDXBz";

        Spell result = CodexTranslator.translate(spell, targetCodex);
        assertEquals(expected, result.getText());
    }

    @Test
    public void testOldAndNewCodexDifferentCharSize(){
        Spell spell = new Spell("Akira Huha huha wangy 91919", oldCodex);

        when(oldCodex.getCharSize()).thenReturn(420);
        when(newCodex.getCharSize()).thenReturn(69);
        Exception exception = assertThrows(IllegalArgumentException.class,
                () -> CodexTranslator.translate(spell, newCodex));
        assertTrue(exception.getMessage().contains("Jumlah karakter pada kedua Codex tidak sama"));
        verify(oldCodex, atLeastOnce()).getCharSize();
        verify(newCodex, atLeastOnce()).getCharSize();
    }

    @Test
    public void testMakeSingleInstanceOfCodexTranslator(){
        CodexTranslator codexTranslator = new CodexTranslator();
        assertTrue(codexTranslator instanceof CodexTranslator);
    }
}
```

## Main Program Tests

```java
@SpringBootTest
public class Tutorial3ApplicationTests {
    @Test
    void contextLoads() {
    }
    @Test
    void testSpringInstanceCanBeMade(){
        Tutorial3Application.main(new String[] {});
    }
}
```