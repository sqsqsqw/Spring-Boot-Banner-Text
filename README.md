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

![Banner接口的实现类全家福]( https://github.com/sqsqsqw/Spring-Boot-Banner-Text/blob/master/Banner接口的实现类全家福jpg)

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

若在resources文件夹中找到至少一个名为banner的文本文档和图片文件，那么将会返回Banners类的对象并分别输出。
若没有，则继续判断fallbackBanner是否为空。


则，SpringApplicationBannerPrinter.getBanner方法是选择对应的Banner实现类