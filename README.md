@Override
public ApplicationDto createApplicationService(ApplicationDto applicationDto) {
    log.info("Start Service createApplication with criteria: {}", applicationDto);

    // Null check for applicationDto
    if (applicationDto == null) {
        throw new IllegalArgumentException("ApplicationDto cannot be null");
    }

    Application application = userMapper.fromDTOToApplication(applicationDto);

    // Null check for application name
    String appName = application.getAppName();
    if (appName == null || appName.isEmpty()) {
        throw new IllegalArgumentException("Application name cannot be null or empty");
    }

    // Check if application with the same name already exists
    Optional<Application> existingApplication = applicationRepository.findApplicationByAppName(appName);
    if (existingApplication.isPresent()) {
        throw new AGPortalException(APP_ALREADY_EXIST);
    }

    application.setDateCreation(LocalDateTime.now());

    // Save the application
    Application savedApplication = applicationRepository.save(application);

    // Null check for saved application
    if (savedApplication == null) {
        throw new RuntimeException("Failed to save application");
    }

    return userMapper.fromApplicationToDTO(savedApplication);
}

 @Test
    public void testCreateApplicationService() {
        // Mock input data
        ApplicationDto inputDto = new ApplicationDto();
        inputDto.setAppName("TestApp");

        // Mock behavior of userMapper
        Application application = new Application();
        when(userMapper.fromDTOToApplication(inputDto)).thenReturn(application);

        // Mock behavior of applicationRepository
        when(applicationRepository.findApplicationByAppName(anyString())).thenReturn(Optional.empty());
        when(applicationRepository.save(any(Application.class))).thenReturn(application);

        // Call the method under test
        ApplicationDto createdDto = yourClassUnderTest.createApplicationService(inputDto);

        // Verify the method behavior
        assertEquals(inputDto.getAppName(), createdDto.getAppName());
        assertEquals(application, userMapper.fromDTOToApplication(inputDto));

        // Verify that repository methods were called with correct arguments
        verify(applicationRepository).findApplicationByAppName(inputDto.getAppName());
        verify(applicationRepository).save(application);
    }

    @Test
    public void testCreateApplicationServiceWithExistingApp() {
        // Mock input data
        ApplicationDto inputDto = new ApplicationDto();
        inputDto.setAppName("ExistingApp");

        // Mock behavior of applicationRepository
        when(applicationRepository.findApplicationByAppName("ExistingApp")).thenReturn(Optional.of(new Application()));

        // Verify that AGPortalException is thrown when the application with the same name already exists
        assertThrows(AGPortalException.class, () -> yourClassUnderTest.createApplicationService(inputDto));

        // Verify that repository methods were called with correct arguments
        verify(applicationRepository).findApplicationByAppName(inputDto.getAppName());
    }

    @Override
    public ApplicationDto updateApplication(ApplicationDto applicationDto) {
        log.info("Start Service update application with criteria: {}", applicationDto);
        Optional<Application> existing = applicationRepository.findApplicationByAppName(applicationDto.getAppName());
        Application application = userMapper.fromDTOToApplication(applicationDto);
        if (!existing.isPresent()) {
            throw new EntityNotFoundException(APP_NOT_FOUND);
        }
        else{
            application.setId(existing.get().getId());
            application.setDateCreation(existing.get().getDateCreation());
        }
        return userMapper.fromApplicationToDTO(applicationRepository.save(application));
    }
}
