package com.socgen.creditbureau.bff.config;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.security.core.context.ReactiveSecurityContextHolder;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationToken;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ServerWebExchange;

import reactor.core.publisher.Mono;

@Component
public class UsernameInjectionFilter implements GlobalFilter {
	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		return ReactiveSecurityContextHolder.getContext().filter(c -> c.getAuthentication() != null).flatMap(c -> {
			JwtAuthenticationToken jwt = (JwtAuthenticationToken) c.getAuthentication();
			String user = (String) jwt.getTokenAttributes().get("email");
			ServerHttpRequest request = exchange.getRequest().mutate().header("x-user", user).build();
			return chain.filter(exchange.mutate().request(request).build());
		}).switchIfEmpty(chain.filter(exchange));
	}
}


package com.socgen.creditbureau.bff.config;

import java.time.Duration;
import java.util.Collections;
import java.util.List;
import java.util.regex.Pattern;

import org.springframework.beans.factory.ObjectProvider;
import org.springframework.cloud.circuitbreaker.resilience4j.ReactiveResilience4JCircuitBreakerFactory;
import org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JConfigBuilder;
import org.springframework.context.annotation.Profile;
import org.springframework.security.config.Customizer;
import org.springframework.cloud.gateway.filter.WebClientHttpRoutingFilter;
import org.springframework.cloud.gateway.filter.WebClientWriteResponseFilter;
import org.springframework.cloud.gateway.filter.headers.HttpHeadersFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.oauth2.client.registration.ReactiveClientRegistrationRepository;
import org.springframework.security.oauth2.client.web.reactive.function.client.ServerOAuth2AuthorizedClientExchangeFilterFunction;
import org.springframework.security.oauth2.client.web.server.ServerOAuth2AuthorizedClientRepository;
import org.springframework.security.web.server.SecurityWebFilterChain;
import org.springframework.web.reactive.function.client.WebClient;

import com.fasterxml.jackson.databind.ObjectMapper;

import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.timelimiter.TimeLimiterConfig;
import org.springframework.web.server.adapter.ForwardedHeaderTransformer;

@EnableWebFluxSecurity
@Configuration
public class Config {

	@Bean
	public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
		http
				.csrf(ServerHttpSecurity.CsrfSpec::disable)
				.authorizeExchange(exchanges -> exchanges
						.pathMatchers("/actuator/**", "/health/**").permitAll()
						.pathMatchers(HttpMethod.OPTIONS).permitAll()
						.anyExchange().authenticated()
				)
				.oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));

		return http.build();
	}

	@Bean
	public WebClientHttpRoutingFilter webClientHttpRoutingFilter(WebClient webClient,
																 ObjectProvider<List<HttpHeadersFilter>> headersFilters) {
		return new WebClientHttpRoutingFilter(webClient, headersFilters);
	}

	@Bean
	public WebClientWriteResponseFilter webClientWriteResponseFilter() {
		return new WebClientWriteResponseFilter();
	}

	@Bean
	// @LoadBalanced
	public WebClient.Builder loadBalancedWebClientBuilder(ReactiveClientRegistrationRepository clientRegistrations,
														  ObjectMapper objectMapper, ServerOAuth2AuthorizedClientRepository clientRepository) {

		ServerOAuth2AuthorizedClientExchangeFilterFunction oauth2ClientFilter = new ServerOAuth2AuthorizedClientExchangeFilterFunction(
				clientRegistrations, clientRepository);
		oauth2ClientFilter.setDefaultClientRegistrationId("bff");
		WebClient.Builder builder = WebClient.builder();
		builder.defaultHeader("Content-Type", MediaType.APPLICATION_JSON.toString());
		builder.defaultHeader("Accept", MediaType.APPLICATION_JSON.toString());
		builder.filter(oauth2ClientFilter);
		return builder;
	}

	@Bean
	WebClient webClient(WebClient.Builder builder) {
		return builder.build();
	}

	/**
	 * Default Resilience4j circuit breaker configuration
	 */
	@Bean
	public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
		return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
				.circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
				.timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(20)).build()).build());
	}

}



server:
  port: 8081
  forward-headers-strategy: framework

spring:
  cloud:
    gateway:
      routes:
        - id: credit-bureau-api
          uri: http://localhost:8085/
          predicates:
            - Path=/api/**
          filters:
            - StripPrefix=1

  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: http://localhost:8080/auth/realms/realm_ce/protocol/openid-connect/certs
          issuer-uri: http://localhost:8080/auth/realms/realm_ce
      client:
        registration:
          bff:
            provider: keycloak
            client-id: credit-bureau-bff
            client-secret: cBXdYjREVy836hIt2WeAeJtLvz9HXR00
            authorization-grant-type: client_credentials
        provider:
          keycloak:
            token-uri: https://apirecette.sgmaroc.root.net/auth/realms/realm-api/protocol/openid-connect/token

logging:
  level:
    org.springframework.security: DEBUG
    org.springframework.web.reactive.function.client: DEBUG

eureka:
  client:
    enabled: false


management:
  endpoints:
    web:
      exposure:
        include: '*'
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.socgen.credibureau</groupId>
	<artifactId>credit-bureau-bff</artifactId>
	<version>1.10.0-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.4.4</version>
		<relativePath/>
	</parent>

	<properties>
		<java.version>17</java.version>
		<spring-cloud.version>2024.0.0</spring-cloud.version>
		<jacoco-maven-plugin.version>0.8.13</jacoco-maven-plugin.version>
		<sonar.projectVersion>0.207.0-SNAPSHOT</sonar.projectVersion>
		<sonar.exclusions>
			**/config/**,
		</sonar.exclusions>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<dependencies>
		<!-- Kubernetes -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-kubernetes-fabric8-all</artifactId>
		</dependency>

		<dependency>
			<groupId>org.junit.jupiter</groupId>
			<artifactId>junit-jupiter</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- Spring Cloud Gateway -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-oauth2-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-loadbalancer</artifactId>
		</dependency>

		<!-- Spring Boot Actuator -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<!-- Resilience4j Circuit Breaker -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
		</dependency>

		<!-- OAuth2 Resource Server -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
		</dependency>

		<!-- OAuth2 Client -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-oauth2-client</artifactId>
		</dependency>

		<!-- Eureka Client -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.cloud</groupId>
					<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.springframework.cloud</groupId>
					<artifactId>spring-cloud-netflix-ribbon</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.netflix.ribbon</groupId>
					<artifactId>ribbon-eureka</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.springframework.cloud</groupId>
					<artifactId>spring-cloud-netflix-hystrix</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<!-- Spring Boot Test -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- Spring Security Test -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- Reactor Test -->
		<dependency>
			<groupId>io.projectreactor</groupId>
			<artifactId>reactor-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- WireMock pour les tests -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-contract-wiremock</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>jakarta.xml.bind</groupId>
			<artifactId>jakarta.xml.bind-api</artifactId>
			<version>4.0.1</version>
		</dependency>

		<dependency>
			<groupId>jakarta.annotation</groupId>
			<artifactId>jakarta.annotation-api</artifactId>
			<version>2.1.1</version>
            <scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>jakarta.validation</groupId>
			<artifactId>jakarta.validation-api</artifactId>
			<version>3.1.1</version>
		</dependency>
		<dependency>
			<groupId>jakarta.servlet</groupId>
			<artifactId>jakarta.servlet-api</artifactId>
			<version>6.0.0</version>
			<scope>provided</scope>
		</dependency>

		<!-- Optionnel : client WireMock pur si tu veux l’utiliser directement -->
		<dependency>
			<groupId>com.github.tomakehurst</groupId>
			<artifactId>wiremock-jre8</artifactId>
			<version>2.35.0</version> <!-- ou version plus récente compatible -->
			<scope>test</scope>
		</dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.openrewrite.maven</groupId>
				<artifactId>rewrite-maven-plugin</artifactId>
				<version>6.15.0</version>
				<configuration>
					<exportDatatables>true</exportDatatables>
					<activeRecipes>
						<recipe>org.openrewrite.java.migrate.UpgradeToJava17</recipe>
					</activeRecipes>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>org.openrewrite.recipe</groupId>
						<artifactId>rewrite-migrate-java</artifactId>
						<version>3.14.1</version>
					</dependency>
				</dependencies>
			</plugin>

			<!-- Spring Boot Plugin -->
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<version>3.4.4</version>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>




			<!-- Jacoco Plugin -->
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>${jacoco-maven-plugin.version}</version>
				<executions>
					<execution>
						<id>prepare-agent</id>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<execution>
						<id>report</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
					<execution>
						<id>post-unit-test</id>
						<phase>test</phase>
						<goals>
							<goal>report</goal>
						</goals>
						<configuration>
							<dataFile>target/jacoco.exec</dataFile>
							<outputDirectory>target/jacoco-ut</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
