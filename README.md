@Override
    public ApplicationDto createApplicationService(ApplicationDto applicationDto)
    {
        log.info("Start Service createApplication with criteria: {}", applicationDto);
        Application application = userMapper.fromDTOToApplication(applicationDto);
        if (applicationRepository.findApplicationByAppName(application.getAppName()).isPresent()) {
            throw new AGPortalException(APP_ALREADY_EXIST);
        }
        application.setDateCreation(LocalDateTime.now());

        return userMapper.fromApplicationToDTO(applicationRepository.save(application));
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
