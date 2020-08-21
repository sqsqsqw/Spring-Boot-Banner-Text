# Spring-Boot-Banner-Text
 Change SpringBoot Project Easter egg

## 这是啥？

下面这个彩蛋表示对于使用过Springboot的Java开发者来说再熟悉不过了。

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)
```
将banner.txt放置在项目中的/src/main/resources目录下，即可将彩蛋更换至文本文档中写的文本。
本repo是用来记录彩蛋是如何生成的，以及有什么好玩的骚操作。

## 深入♂了解

我们可以在 SpringApplication.run(String.. args) 方法中找到相关的代码。 
```java
	public ConfigurableApplicationContext run(String... args) {
...
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);		//this
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
...
	}
```

继续深入了解一下。
```java
// SpringApplication.class
...
	private Banner printBanner(ConfigurableEnvironment environment) {
		if (this.bannerMode == Banner.Mode.OFF) {
			return null;
		}
		ResourceLoader resourceLoader = (this.resourceLoader != null)
				? this.resourceLoader : new DefaultResourceLoader(getClassLoader());
		SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(
				resourceLoader, this.banner);
		if (this.bannerMode == Mode.LOG) {
			return bannerPrinter.print(environment, this.mainApplicationClass, logger);
		}
		return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
	}
...

// SpringApplicationBannerPrinter.class
...
	public Banner print(Environment environment, Class<?> sourceClass, PrintStream out) {
		Banner banner = getBanner(environment);
		banner.printBanner(environment, sourceClass, out);
		return new PrintedBanner(banner, sourceClass);
	}
	private Banner getBanner(Environment environment) {
		Banners banners = new Banners();
		banners.addIfNotNull(getImageBanner(environment));
		banners.addIfNotNull(getTextBanner(environment));
		if (banners.hasAtLeastOneBanner()) {
			return banners;
		}
		if (this.fallbackBanner != null) {
			return this.fallbackBanner;
		}
		return DEFAULT_BANNER;
	}
...

// SpringBootBanner.class
class SpringBootBanner implements Banner {

	private static final String[] BANNER = { "",
			"  .   ____          _            __ _ _",
			" /\\\\ / ___'_ __ _ _(_)_ __  __ _ \\ \\ \\ \\",
			"( ( )\\___ | '_ | '_| | '_ \\/ _` | \\ \\ \\ \\",
			" \\\\/  ___)| |_)| | | | | || (_| |  ) ) ) )",
			"  '  |____| .__|_| |_|_| |_\\__, | / / / /",
			" =========|_|==============|___/=/_/_/_/" };

	private static final String SPRING_BOOT = " :: Spring Boot :: ";

	private static final int STRAP_LINE_SIZE = 42;

	@Override
	public void printBanner(Environment environment, Class<?> sourceClass,
			PrintStream printStream) {
		for (String line : BANNER) {
			printStream.println(line);
		}
		String version = SpringBootVersion.getVersion();
		version = (version != null) ? " (v" + version + ")" : "";
		StringBuilder padding = new StringBuilder();
		while (padding.length() < STRAP_LINE_SIZE
				- (version.length() + SPRING_BOOT.length())) {
			padding.append(" ");
		}

		printStream.println(AnsiOutput.toString(AnsiColor.GREEN, SPRING_BOOT,
				AnsiColor.DEFAULT, padding.toString(), AnsiStyle.FAINT, version));
		printStream.println();
	}

}

```
慢慢解读这些代码。

### SpringApplication.printBanner做了什么

首先Banner.Mode为枚举类，存在'Banner.Mode.OFF", "Banner.Mode.CONSOLE" 和 "Banner.Mode.LOG" 三种模式。
```java
//Banner.class
	enum Mode {

		/**
		 * Disable printing of the banner.
		 */
		OFF,

		/**
		 * Print the banner to System.out.
		 */
		CONSOLE,

		/**
		 * Print the banner to the log file.
		 */
		LOG

	}

```
bannerMode为SpringApplication的属性，默认为Banner.Mode.CONSOLE。	
我们同时可以在调用SpringApplication.run前使用SpringApplication.setBannerMode方法进行更改
```java
//SpringApplication.class
	/**
	 * Sets the mode used to display the banner when the application runs. Defaults to
	 * {@code Banner.Mode.CONSOLE}.
	 * @param bannerMode the mode used to display the banner
	 */
	public void setBannerMode(Banner.Mode bannerMode) {
		this.bannerMode = bannerMode;
	}
```

那么printBanner从总体来看就是对输出进行判断，判断是否需要进行输出，以及使用什么输出流进行输出。
在写一遍源码。
```java
// SpringApplication.class
...
	private Banner printBanner(ConfigurableEnvironment environment) {
		if (this.bannerMode == Banner.Mode.OFF) {
			return null;
		}
		ResourceLoader resourceLoader = (this.resourceLoader != null)
				? this.resourceLoader : new DefaultResourceLoader(getClassLoader());
		SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(
				resourceLoader, this.banner);
		if (this.bannerMode == Mode.LOG) {
			return bannerPrinter.print(environment, this.mainApplicationClass, logger);
		}
		return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
	}
...

```
### SpringApplicationBannerPrinter.print做了什么
pringApplicationBannerPrinter类中的私有方法getBanner使用Banner接口的实现类SpringBootBanner对Banner进行输出。
而getBanner方法中使用到的Banners是实现了Banner接口的将实现Banner接口的对象保存在ArrayList中的封装内部类。（禁止套娃）
```java
//SpringApplicationBannerPrinter.class
  private static class Banners implements Banner {

    private final List<Banner> banners = new ArrayList<>();

    public void addIfNotNull(Banner banner) {
      if (banner != null) {
        this.banners.add(banner);
      }
    }

    public boolean hasAtLeastOneBanner() {
      return !this.banners.isEmpty();
    }

    @Override
    public void printBanner(Environment environment, Class<?> sourceClass,
        PrintStream out) {
      for (Banner banner : this.banners) {
        banner.printBanner(environment, sourceClass, out);
      }
    }

  }
```

我们先来了解一下实现了Banner接口的类都有哪些。
来一张全家福。
![Banner接口的实现类]( https://github.com/sqsqsqw/Spring-Boot-Banner-Text/blob/master/BannerBanner接口的实现类.jpg)

![Banner接口的实现类全家福]( https://github.com/sqsqsqw/Spring-Boot-Banner-Text/blob/master/Banner接口的实现类全家福.jpg)

Banners, ImageBanner, PrintedBanner, ResourceBanner, SpringBootBanner。
其中，Banners类已经有所接触，而其他的类则是用于Banner输出的Banner实现类。

再次将源码搬过来。
```java
// SpringApplicationBannerPrinter.class
  private Banner getBanner(Environment environment) {
    Banners banners = new Banners();
    banners.addIfNotNull(getImageBanner(environment));
    banners.addIfNotNull(getTextBanner(environment));
    if (banners.hasAtLeastOneBanner()) {
      return banners;
    }
    if (this.fallbackBanner != null) {
      return this.fallbackBanner;
    }
    return DEFAULT_BANNER;
  }

```
我们可以看到有两行代码
```java
    banners.addIfNotNull(getImageBanner(environment));
    banners.addIfNotNull(getTextBanner(environment));
```
而其中的getImageBanner和getTextBanner则分别使用了ImageBanner和ResourceBanner
```java
//SpringApplicationBannerPrinter.class
  private Banner getTextBanner(Environment environment) {
    String location = environment.getProperty(BANNER_LOCATION_PROPERTY,
        DEFAULT_BANNER_LOCATION);
    Resource resource = this.resourceLoader.getResource(location);
    if (resource.exists()) {
      return new ResourceBanner(resource);
    }
    return null;
  }

  private Banner getImageBanner(Environment environment) {
    String location = environment.getProperty(BANNER_IMAGE_LOCATION_PROPERTY);
    if (StringUtils.hasLength(location)) {
      Resource resource = this.resourceLoader.getResource(location);
      return resource.exists() ? new ImageBanner(resource) : null;
    }
    for (String ext : IMAGE_EXTENSION) {
      Resource resource = this.resourceLoader.getResource("banner." + ext);
      if (resource.exists()) {
        return new ImageBanner(resource);
      }
    }
    return null;
  }
```
其中，getTextBanner中的DEFAULT_BANNER_LOCATION则是banner.txt，getImageBanner中的IMAGE_EXTENSION是{ "gif", "jpg", "png" }
```java
//SpringApplicationBannerPrinter.class
  static final String BANNER_LOCATION_PROPERTY = "spring.banner.location";

  static final String BANNER_IMAGE_LOCATION_PROPERTY = "spring.banner.image.location";

  static final String DEFAULT_BANNER_LOCATION = "banner.txt";

  static final String[] IMAGE_EXTENSION = { "gif", "jpg", "png" };
```

若在resources文件夹中找到至少一个名为banner的文本文档或图片文件，那么将会返回Banners类的对象并分别输出。
若没有，则继续判断fallbackBanner是否为空。若不为空，则使用fallbackBanner，若为空，则使用SpringBootBanner类。
fallbackBanner可以通过SpringApplicationBannerPrinter的构造函数进行赋值，而SpringApplication类的printBanner方法将this.banner作为参数传入。
SpringApplication的banner默认为空NULL，但是可以通过setter进行赋值。
则，SpringApplicationBannerPrinter.getBanner方法是选择对应的Banner实现类。

Banner接口类有printBanner方法，所有实现了Banner接口的类都需要实现这个方法，则彩蛋就需要调用printBanner方法进行输出。而SpringBootBanner类是SpringBoot默认的Banner输出类。

## 总结
SpringBoot默认有SpringBootBanner类可以进行默认输出，但可以通过在resources文件夹下创建文件名为banner的.txt/.jpg/.png/.gif文件对默认输出进行修改，而且多文件可以同时输出。突然发现可以输出gif这么好玩的事情，怎么能不尝试一下呢。但当我兴高采烈地将gif文件放进工程文件并看到接下来的东西，，，
```

  :::o:&::::::::::::::888&&oo&ooooooooooo:ooo#&&&&&&&&&&&&#::*@@@@@@8#########
  :::o88o:::::::::::::888&&oooooooooo::::::::o#&&&&&&&&&::*:::::8@@@@#########
  :::88#8:::::::::::::888&&ooooooo:::::::::::ooooooooo&:::::::::oo@@@@########
  8@88#8##&:::::::::::888&&oooo:o:::::::::::::o#oooooo&oo####oooo&#@@@@#######
  #88#88#8##8:::::::::888&&ooo::::::::::::::::oo&ooooo&o8&8&oo&&&88@@@@#######
  8##@88@###8:::::::::888&&ooo:::::::::::::::::o8oooooo&&&&oo8&&8#@@@@@#######
  ####8#@####&::::::o8888&oooo::::::::::o:::o:ooo8oooo&@@88#@@@@@@@@#@@#######
  #@@88#@@###&::::::::888&&ooo::::::::::o:::::ooo8oooo&@@@@@@@@@@@@@#@@#######
  @##8##@@###8::::::::888&oooooo::::::::o:o::ooooo@ooo8@@@@@@@@@@@@#8@@#######
  8#8#@@@@@##8o:::::::888&oooo::::::::::ooooooooo&&888o:*::o8##@@@@@@@@#######
  :88#8@#@@###o:::::::88&&oooo:::::::::::oooo&8&@#8###:&:*o8#@@@@@@@@@@#######
  :#@#@@#@#8#8&:::::::888&oooo::::::::::::&######@8##8*&o:&8@@@@@@@@@@@@######
  ::8####@8&&88:::::::88&&oooooo&&&o&o:::8@@@@@@@@###8*&o:&8@@@@@@@@@@@@######
  :::&88888&oo&o::::::88&&ooo&&&&&&&&&&o8@@@@@@@@@@#88o&o88#@@@@@@@@@@@@######
  ::::o&&&8&o:::::::::888&oo&&&&&&&&&88&@@@@@@@@@@@#88*o*:o8@@@@@@@##@@@######
  ::::::o&&oo:::::::::888&&888#@@88@#@###@@@@@@@@@@@#8:8:*8#@@@@@@@@@@@@######
  ::::::::::o:::::::::888&88#88&&&8&&88###@@@@@@@@@@@#o8888#@@@@@@@@@@@@######
  ::::::::::::::::::::888888#@###@@#@@@8&8#@@@@@@@@@@@###8#@@@@@@@@@@@@@@#####
  ::::::::::::::::::::888###@@@@@@@@@@@#8&&8#@@@@@@@@@@@####@@@@@@@@@@@@@#####
  ::::::::::::::::::::888##@@#&&8###&&&o&88&&8#@@@@@@@@@###@@@@@@@@@@@@@######
  o:::::::::::::::::::8888&88&&ooooooo&oo&8&&&8@@@@@@@@@@####@@@@@@@@@@@######
  o:::::::::::::::::::888&&&&&ooo&ooo&&&88&&&oo&@@@@@@@@@@##8#@@@@@@@@@@######
  o:::::::::::::::::::888&&o&&&&&8o&&&8##8888&&&@@@@@@@@@@@@@@###@#@@@@#######
  &o::::::::::::::::::888&&&&&&8888&88&88&&&8&&&@@@@@@@@@@@@@@@@@@@@@@@@######
  8o::::::::::::::::::888&&&&8&&&&&8888&&&&&8888@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
  8o::::::::::::::::::8888&&&&&&o&&88888&o&8@#8#@@@@@&#@@@@@@@@@@@@@@@@@@@@@@@
  8ooo::::::::::::::::888&&&o888&&88#@#88###@@#@@@@@@###@@@@@@@@@@@@@@@@@##@@@
  #&o:::::::::::::::::888&&oo&8@@@@@@@@88#@@@@@@@@@@@&@@@@@@@@@@@@@@@@@@@#####
  @&oo::::::::::::::::8#8&&oo&88#888@##88@@@@@@@@@@@#8@@#8@@#o@@@@@@@@@@@@@@@#
  @&ooo:::::::::::::::8##8&oo&&888@@##8#@@@@@@@@@@@@@&###@@@#&@@@@@@@@@@@@@@@@
  @&&oo:::::::::::::::8@#8&oo&&&&8##88&8##@@@@@@@@@@@@@@@@@#8o@@@@@@@@@@@@@@@@
  @8&oo::::::::::::::o8@##&&o&&&&888####88#@@@@@@@@@@@@@@@@@@o@@@@@@@@@@@@@@@@
  @8&ooo::::::::::oooo8@@#8&&&&&&&8###8&&&&@@@@@@@@@@@@@@@@@@o@@@@@@@@@@@@@@@@
  @8&oooo:::::::o:oooo8@@@#8&&&&&88##8&&&88#@@@@@@@@@@@@@@@@@&@@@@@@@@@@@@@@@@
  @8&&ooooo:oo::o:oooo8@@@8&&&&8&88#@#&&&&&#@@@@@@##@@@@@@@@@&@@@@@@@@@@@@@@@@
  @&&&oooooooooo:::ooo8@@@#88888&88@@#8&&88#@@@@@@@@@@@@@@@@@8@@@@@@@@@@@@@@@@
  @&&&ooooo:::oooooooo8@@@#888&&88@@@#88888#@@@@@@@@@@@@@@@@@8@@@@@@@@@@@@@@@@
  @&&&ooooo:oooooooooo8@@@#8888888@@@#8888#@@@@@@@@@#@@@@@@@@8@@@@@@@@@@@@@@@@
  @&o&ooooo:oooooooooo8@@@#8888888@@@#&&&#@@@@@@@@@@@@@@@@@@@8@@@@@@@@@@@@@@@@
  @&&&oooooooooooooooo&@@@#8888888#@@888#@@@@@@@@@@@@##@@@@@@#@@@@@@@@@@@@@@@@
  @&&&oooooooooooooooo8@@@#88888888@@88&8#@@@#&8#@@@@@@@@@@@@##@@@@@@@@@@@@@@@
  @&&ooooooooo:ooooooo8@@@@#8888&8#@##@@#@@#888888#@#&&&#@@@@8#@@@@@@@@@@@@@@@
  @8&&oooooooo:ooooooo8@@88&&8@#@8&&&&&&88888888888888888#@@@o8@@@@@@@@@@@@@@@
  @88&&&oooooooooooooo88&&&&&&&&&&&&&&&&&&&&&&&&888888888888888888888888#@@@@@
  @88&&&oooo&888&&oooo&&&&&&&&&&&&&&&&&&&&&&&&&&&888&&888888888888888888888#@@
  @@@#8&&ooooooooooo&oo&&&&oooooo&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&88888&8&888888#
  @#88&&&&&ooooooooooo&&&&&oo&&oo&&&&&&&&&&&&&&&&&&&&&&&8&&&&&&&&&&&&&&88&8888
  #&&&&&oooooooooooooo&&&&&&oooooo&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&88&&888
  &&&&oooooooooooooooooo&&&&&ooooo&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&88888
  &&&oooooooooooooooooo&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&88888888888
  &&&ooooooo&&oooooooooo&&&&&&&&&&&&&&&ooo&&&&&&&&&&&&&&&&&&&&&&&&&88888888#8#
  &&ooooooooooooooooooooooooo&&&&&&&&&o&&&&&&&&&&&&&&&&&&&&&&&&&&&&&88888#####
  &&&&&&oooooooooooooooo&&&o&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&8888###@@

By
 _   _           _          [=]
| |_| | __ _ ___| |__   __ _ _
|  _  |/ _` / __| '_ \ / _` | |
| | | | (_| \__ \ | | | (_| | |
\_| |_/\__,_|___/_| |_|\__, |_|
                          |_|

```
希望大家能看得出来这是一只狗吧。
（output.txt可以查看所有的输出文件）