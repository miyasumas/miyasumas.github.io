# Spring Bootの設定値をMapで受けたい

## 遭遇したできごと

当然のように↓みたいにしたら、エラー

```yaml
config:
  map:
    key1: value1
    key2: value2
```

```java
@Configuration
public class Config {

  @Value("${config.map}")
  private Map<String, String> map;
}
```

エラー発生。

```
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'config.map' in value "${config.map}"
	at org.springframework.util.PropertyPlaceholderHelper.parseStringValue(PropertyPlaceholderHelper.java:178)
	at org.springframework.util.PropertyPlaceholderHelper.replacePlaceholders(PropertyPlaceholderHelper.java:124)
	at org.springframework.core.env.AbstractPropertyResolver.doResolvePlaceholders(AbstractPropertyResolver.java:236)
	at org.springframework.core.env.AbstractPropertyResolver.resolveRequiredPlaceholders(AbstractPropertyResolver.java:210)
	at org.springframework.context.support.PropertySourcesPlaceholderConfigurer.lambda$processProperties$0(PropertySourcesPlaceholderConfigurer.java:175)
	at org.springframework.beans.factory.support.AbstractBeanFactory.resolveEmbeddedValue(AbstractBeanFactory.java:908)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1228)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1207)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:636)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:116)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:397)
	... 155 more
```

エラー内容見ても何がなんだかわからず…。

## 正解

`@ConfigurationProperties` を使って `setter` を付けておくのが正解らしい。動いた。

```
@ConfigurationProperties("config")
public class Config {

  private Map<String, String> map;

  public void setMap(Map<String, String> map) {
    this.map = map;
  }
}
```

`@ConfigurationProperties` と `@Value` は動きが違って混乱した。

