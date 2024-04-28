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
