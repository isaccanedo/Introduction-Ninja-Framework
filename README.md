### Relevant Articles

# 1. Visão Geral
Hoje em dia, existem muitos frameworks baseados em JEE, como Spring, Play e Grails, disponíveis para o desenvolvimento de aplicativos da web.

Podemos ter nossos motivos para escolher um deles em vez dos outros. No entanto, nossa escolha também depende do caso de uso e do problema que estamos tentando resolver.

Neste tutorial introdutório, exploraremos a estrutura da web Ninja e criaremos um aplicativo da web simples. Ao mesmo tempo, examinaremos alguns dos recursos básicos que ele fornece.

# 2. Ninja
Ninja é uma estrutura da web full-stack, porém leve, que utiliza bibliotecas Java existentes para realizar o trabalho.

Tendo recursos de HTML para renderização JSON, persistência para teste, é uma solução completa para a construção de aplicativos da web escaláveis.

Ele segue o paradigma da convenção sobre a configuração e categoriza o código em pacotes como modelos, controladores e serviços.

Ninja usa bibliotecas Java populares para recursos-chave como Jackson para renderização JSON / XML, Guice para gerenciamento de dependências, Hibernate para persistência e Flyway para migrações de banco de dados.

Para um desenvolvimento rápido, ele oferece SuperDevMode para recarregamento a quente do código. Portanto, permite-nos ver as mudanças instantaneamente no ambiente de desenvolvimento.

# 3. Configuração
O Ninja requer um conjunto padrão de ferramentas para criar um aplicativo da web:

- Java 1.8 ou posterior;
- Maven 3 ou posterior;
- IDE (Eclipse ou IntelliJ).

Usaremos um arquétipo Maven para configurar o projeto Ninja rapidamente. Ele solicitará que forneçamos um id de grupo, um id de artefato e um número de versão, seguido por um nome de projeto:

Usaremos um arquétipo Maven para configurar o projeto Ninja rapidamente. Ele solicitará que forneçamos um id de grupo, um id de artefato e um número de versão, seguido por um nome de projeto:

```
mvn archetype:generate -DarchetypeGroupId=org.ninjaframework \
  -DarchetypeArtifactId=ninja-servlet-archetype-simple
```

Ou, para um projeto Maven existente, podemos adicionar a dependência ninja-core mais recente ao pom.xml:

```
<dependency>
    <groupId>org.ninjaframework</groupId>
    <artifactId>ninja-core</artifactId>
    <version>6.5.0</version>
</dependency>
```

Em seguida, executaremos o comando Maven para compilar os arquivos pela primeira vez:

```
mvn clean install
```

Por último, vamos executar o aplicativo usando um comando Maven fornecido pelo Ninja:

```
mvn ninja:run
```

Voila! Nosso aplicativo foi iniciado e estará acessível em localhost:8080:

# 4. Estrutura do Projeto
Vamos dar uma olhada na estrutura do projeto semelhante ao Maven criada pelo Ninja:

A estrutura cria alguns pacotes com base em convenções.

As classes Java são categorizadas em diretórios conf, controllers, models e services em src/main/java.

Da mesma forma, src/test/java contém as classes de teste de unidade correspondentes.

O diretório views em src/main/java contém os arquivos HTML. E o diretório src/main/java/assets contém recursos como imagens, folhas de estilo e arquivos JavaScript.

# 5. Controlador
Estamos prontos para discutir alguns recursos básicos da estrutura. Um controlador é uma classe que recebe uma solicitação e retorna a resposta com resultados específicos.

Primeiro, vamos discutir algumas convenções a seguir:

- Crie uma classe no pacote de controladores e sufixe o nome com Controller;
- Um método que atende à solicitação deve retornar o objeto da classe Result.
Vamos criar a classe ApplicationController com um método simples para renderizar o HTML:

```
@Singleton
public class ApplicationController {
    public Result index() {
        return Results.html();
    }
}
```

Aqui, o método index irá renderizar um HTML chamando o método html da classe Results. O objeto Result contém tudo o que é necessário para renderizar o conteúdo, como código de resposta, cabeçalhos e cookies.

Nota: a anotação @Singleton de Guice permite apenas uma instância do controlador em todo o aplicativo.

# 6. Ver
Para o método de índice, o Ninja procurará o arquivo HTML - index.ftl.html no diretório 
views/ApplicationController.

O Ninja usa o mecanismo de modelo Freemarker para renderização de HTML. Portanto, todos os arquivos em visualizações devem ter a extensão .ftl.html.

Vamos criar o arquivo index.ftl.html para o método index:

```
<html>  
<head>
    <title>Ninja: Index</title>
</head>
<body>
    <h1>${i18n("helloMsg")}</h1>
    <a href="/userJson">User Json</a>
</body>
</html>
```

Aqui, usamos a tag i18n fornecida pelo Ninja para obter a propriedade helloMsg do arquivo message.properties. Discutiremos isso mais adiante na seção de internacionalização.

# 7. Rota
A seguir, definiremos a rota para a solicitação chegar ao método de índice.

O Ninja usa a classe Routes no pacote conf para mapear uma URL para um método específico do controlador.

Vamos adicionar uma rota para acessar o método de índice do ApplicationController:

```
public class Routes implements ApplicationRoutes {
    @Override
    public void init(Router router) {          
        router.GET().route("/index").with(ApplicationController::index);
    }
}
```

É isso! Estamos prontos para acessar a página de índice em localhost:8080/index:

# 8. Renderização JSON
Como já discutido, Ninja usa Jackson para renderização JSON. Para renderizar o conteúdo JSON, podemos usar o método json da classe Results.

Vamos adicionar o método userJson na classe ApplicationController e renderizar o conteúdo de um HashMap simples em JSON:

```
public Result userJson() {
    HashMap<String, String> userMap = new HashMap<>();
    userMap.put("name", "Norman Lewis");
    userMap.put("email", "norman@email.com");    
    return Results.json().render(user);
}
```

Em seguida, adicionaremos o roteamento necessário para acessar o userJson:

```
router.GET().route("/userJson").with(ApplicationController::userJson);
```

Agora, podemos renderizar JSON usando localhost:8080/userJson:


# 9. Serviço

Podemos criar um serviço para manter a lógica de negócios separada do controlador e injetar nosso serviço sempre que necessário.

Primeiro, vamos criar uma interface UserService simples para definir a abstração:

```
public interface UserService {
    HashMap<String, String> getUserMap();
}
```

Em seguida, implementaremos a interface UserService na classe UserServiceImpl e substituiremos o método getUserMap:

```
public class UserServiceImpl implements UserService {
    @Override
    public HashMap<String, String> getUserMap() {
        HashMap<String, String> userMap = new HashMap<>(); 
        userMap.put("name", "Norman Lewis"); 
        userMap.put("email", "norman@email.com"); 
        return userMap;
    }
}
```

Em seguida, ligaremos a interface UserService à classe UserServiceImpl usando o recurso de injeção de dependência do Ninja fornecido pelo Guice.

Vamos adicionar a vinculação na classe Module disponível no pacote conf:

```
@Singleton
public class Module extends AbstractModule {
    protected void configure() {        
        bind(UserService.class).to(UserServiceImpl.class);
    }
}
```

Por último, injetaremos a dependência UserService na classe ApplicationController usando a anotação @Inject:

```
public class ApplicationController {
    @Inject
    UserService userService;
    
    // ...
}
```

Portanto, estamos prontos para usar o método getUserMap do UserService no ApplicationController:

```
public Result userJson() {
    HashMap<String, String> userMap = userService.getUserMap();
    return Results.json().render(userMap);
}
```

# 10. Flash Scope
O Ninja oferece uma maneira simples, porém eficiente, de lidar com mensagens de sucesso e erro de solicitações por meio de seu recurso chamado Flash Scope.

Para usá-lo no controlador, adicionaremos o argumento FlashScope ao método:

```
public Result showFlashMsg(FlashScope flashScope) {
    flashScope.success("Success message");
    flashScope.error("Error message");
    return Results.redirect("/home");
}
```

### Nota: O método de redirecionamento da classe Results redireciona o destino para o URL fornecido.

Em seguida, adicionaremos um routing/flash ao método showFlashMsg e modificaremos a visualização para mostrar as mensagens flash:

```
<#if (flash.error)??>
    <div class="alert alert-danger">
        ${flash.error}
    </div>
</#if>
<#if (flash.success)??>
    <div class="alert alert-success">
        ${flash.success}
    </div>
</#if>
```

Agora, podemos ver o FlashScope em ação em localhost:8080/flash:


# 11. Internacionalização
O Ninja oferece um recurso de internacionalização integrado que é fácil de configurar.

Primeiro, definiremos a lista de idiomas suportados no arquivo application.conf:

```
application.languages=fr,en
```

Em seguida, criaremos o arquivo de propriedades padrão - messages.properties para inglês - com pares de valores-chave para mensagens:

```
header.home=Home!
helloMsg=Hello, welcome to Ninja Framework!
```

Da mesma forma, podemos adicionar o código do idioma no nome do arquivo para um arquivo de propriedades específico do idioma - por exemplo, arquivo message_fr.properties para francês:

```
header.home=Accueil!
helloMsg=Bonjour, bienvenue dans Ninja Framework!
```

Assim que as configurações estiverem prontas, podemos habilitar facilmente a internacionalização na classe ApplicationController.

Temos duas maneiras, usando a classe Lang ou a classe Mensagens:

```
@Singleton
public class ApplicationController {
    @Inject
    Lang lang;

    @Inject
    Messages msg;
    
    // ...
}
```

Então, usando a classe Lang, podemos definir a linguagem do resultado:

```
Result result = Results.html();
lang.setLanguage("fr", result);
```

Da mesma forma, usando a classe Mensagens, podemos obter uma mensagem específica do idioma:

```
Optional<String> language = Optional.of("fr");        
String helloMsg = msg.get("helloMsg", language).get();
```

# 12. Persistência
O Ninja oferece suporte a JPA 2.0 e utiliza o Hibernate para permitir a persistência no aplicativo da web. Além disso, oferece suporte de banco de dados H2 integrado para rápido desenvolvimento.

### 12.1. Modelo
Exigimos uma classe Entity para conectar a uma tabela no banco de dados. Para isso, o Ninja segue a convenção de procurar as classes de entidade no pacote de modelos. Então, vamos criar a classe de entidade User lá:

```
@Entity
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    Long id;
    public String firstName;
    public String email;  
}
```

Em seguida, configuraremos o Hibernate e definiremos os detalhes para a conexão do banco de dados.

### 12.2. Configuração
Para a configuração do Hibernate, o Ninja espera que o arquivo persistence.xml esteja 
no diretório src/main/java/META-INF:

```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
  version="2.0">
   
    <!-- Database settings for development -->
    <persistence-unit name="dev_unit"
      transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <property name="hibernate.connection.driver_class" value="org.h2.Driver" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
            <property name="hibernate.connection.autocommit" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```

Em seguida, adicionaremos os detalhes de conexão do banco de dados ao application.conf:

```
ninja.jpa.persistence_unit_name=dev_unit
db.connection.url=jdbc:h2:./devDb
db.connection.username=sa
db.connection.password=
```

### 12.3. EntityManager
Por último, injetaremos a instância do EntityManager no ApplicationController usando a classe Provider do Guice:

```
public class ApplicationController {
    @Inject 
    Provider<EntityManager> entityManagerProvider;

    // ...
}
```

Portanto, estamos prontos para usar o EntityManager para persistir o objeto User:

```
@Transactional
public Result insertUser(User user) {
    EntityManager entityManager = entityManagerProvider.get();
    entityManager.persist(user);
    entityManager.flush();
    return Results.redirect("/home");
}
```

Da mesma forma, podemos usar o EntityManager para ler o objeto User do DB:

```
@UnitOfWork
public Result fetchUsers() {
    EntityManager entityManager = entityManagerProvider.get();
    Query q = entityManager.createQuery("SELECT x FROM User x");
    List<User> users = (List<User>) q.getResultList();
    return Results.json().render(users);
}
```

Aqui, a anotação @UnitOfWork do Ninja tratará de tudo sobre as conexões de banco de dados sem lidar com transações. Portanto, pode ser útil para consultas somente leitura, onde geralmente não exigimos transações.

# 13. Validação
O Ninja fornece suporte integrado para validações de bean seguindo as especificações JSR303.

Vamos examinar o recurso anotando uma propriedade na entidade User com a anotação @NotNull:

```
public class User {
    // ...
    
    @NotNull
    public String firstName;
}
```

Em seguida, modificaremos o método insertUser já discutido no ApplicationController para habilitar a validação:

```
@Transactional
public Result insertUser(FlashScope flashScope, @JSR303Validation User user, Validation validation) {
    if (validation.getViolations().size() > 0) {
        flashScope.error("Validation Error: User can't be created");
    } else {
        EntityManager entityManager = entitiyManagerProvider.get();
        entityManager.persist(user);
        entityManager.flush();
        flashScope.success("User '" + user + "' is created successfully");
    }
    return Results.redirect("/home");
}
```

Usamos a anotação @JSR303Validation do Ninja para habilitar a validação do objeto User. Em seguida, adicionamos o argumento Validation para trabalhar com validações por meio de métodos como hasViolations, getViolations e addViolation.

Por último, o objeto FlashScope é usado para mostrar o erro de validação na tela.

Nota: Ninja segue as especificações JSR303 para validações de bean. No entanto, a especificação JSR380 (Bean Validation 2.0) é o novo padrão.

# 14. Conclusão
Neste artigo, exploramos a estrutura da web Ninja - uma estrutura de pilha completa que fornece recursos úteis usando bibliotecas Java populares.

Para começar, criamos um aplicativo da web simples usando controladores, modelos e serviços. Em seguida, ativamos o suporte JPA no aplicativo para persistência.

Ao mesmo tempo, vimos alguns recursos básicos como Rotas, renderização JSON, Internacionalização e Escopos Flash.

Por último, exploramos o suporte de validação fornecido pelo framework.