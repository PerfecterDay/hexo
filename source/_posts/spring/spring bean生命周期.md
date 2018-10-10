---
title: spring bean 的生命周期
date: 2018-09-12  11:12:34
tags: spring            
category: spring
---

# BeanFactory中bean的生命周期

![BeanFactory 中bean的生命周期](/pics/beanfactory-bean-lifecycle.png)

BeanFactory 实例化一个 `bean` 的步骤：

![BeanFactory 中bean的生命周期](/pics/spring-bean.JPG)

注意到：自定义的 `bean` 实例化时，会调用 `BeanPostProcessor`相关的方法，`BeanPostProcessor` 不一定是 `bean` 本身实现的接口，它是 Spring 容器提供的容器级别接口，所有实现 `BeanPostProcessor` 的类在注册到容器中后，都会在 `spring` 实例化某个`bean`的时候其作用。就是说，如果我们自定义了一个 `BeanPostProcessor` 的实现类并注册到容器中，则它会在`spring`实例化所有其它`bean`的时候起作用。

# ApplicationContext中bean的生命周期

![ApplicationContext 中bean的生命周期](/pics/applicationContext-bean-lifecycle.png)

Bean 在应用上下文中的生命周期与在 BeanFactory 中的类似，实际上，应用上下文在初始化时，会向容器中注册一个 `ApplicationContextAwareProcessor` 类型的 `BeanPostProcessor` 实现类，并实现 `postProcessBeforeInitialization` 方法，这样的话，调用 `getBean`方法时， `ApplicationContextAwareProcessor` 就会生效， `postProcessBeforeInitialization` 判断如果 Bean 实现了 `EnvironmentAware` , `EmbeddedValueResolverAware` , `ResourceLoaderAware` , `ApplicationEventPublisherAware` , `MessageSourceAware` , `ApplicationContextAware` 接口，则会分别调用它们。

ApplicationContext 和 BeanFactory 的一个重大区别在于：前者会利用java反射机制自动识别出注册的 `BeanPostProcessor`, `BeanFactoryPostProcessor` , `InstantiationAwareBeanPostProcessor` ,并自动将它们注册到容器中；而后者需要手工调用 `addBeanPostProcessor` 方法。

`ApplicationContext` 的 `refresh` 方法，会在 `ApplicationContext` 初始化时调用：

	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// 获取beanFactory，此时会解析加载xml中的beanDefinition，但是并没有注册 bean 对象
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // 注册一些 BeanPostProcessor bean，开始生成 bean
			prepareBeanFactory(beanFactory);

            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // 注册并调用BeanFactoryPostProcessor的postProcessBeanFactory方法
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册 BeanPostProcessor 类型的bean，自动实现 BeanPostProcessor 注册
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
    }

看看几个主要的方法：
1. `obtainFreshBeanFactory`:

    protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}

    @Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);
        //加载beanDefinition
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
	}
该方法主要是实例化一个 `beanFactory`，并加载 bean 定义。

2. `prepareBeanFactory`:

    protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}

3. `registerListeners`:

    protected void registerListeners() {
		// Register statically specified listeners first.
		for (ApplicationListener<?> listener : getApplicationListeners()) {
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}

		// Publish early application events now that we finally have a multicaster...
		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
		this.earlyApplicationEvents = null;
		if (earlyEventsToProcess != null) {
			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
				getApplicationEventMulticaster().multicastEvent(earlyEvent);
			}
		}
	}