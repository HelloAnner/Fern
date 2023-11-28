```java
ExecutorService executorService = Executors.newFixedThreadPool(10);  
CompletionService<User> completionService = new ExecutorCompletionService<>(executorService);  
  
IntStream.range(0, 51000).forEach(i -> {  
    completionService.submit(() -> {  
        try {  
            String uuid = UUIDUtil.generate();  
            User user = new User().id(uuid)  
                    .userName("user" + uuid)  
                    .realName("user" + i)  
                    .creationType(ManualOperationType.KEY)  
                    .lastOperationType(ManualOperationType.KEY)  
                    .password(SecurityToolbox.sha256("1"))  
                    .enable(true);  
            PlatformScaffoldConfiguration.getPlatformScaffoldDBProvider().getAuthorityContextProvider().getUserController().add(user);  
            FineLoggerFactory.getLogger().info("count : {}", i);  
            return user;  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
    });  
});  
  
List<CompletableFuture<User>> futures = new ArrayList<>(51000);  
for (int i = 0; i < 51000; i++) {  
    futures.add((CompletableFuture<User>) completionService.take());  
}  
  
List<User> results = new ArrayList<>(51000);  
for (CompletableFuture<User> future : futures) {  
    try {  
        results.add(future.get());  
    } catch (InterruptedException | ExecutionException e) {  
        throw new RuntimeException(e);  
    }  
}  
  
executorService.shutdown();
```

