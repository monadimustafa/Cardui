public List<ApplicationDto> getRequests(List<ApplicationDto> applicationsDto, String email)
    {
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        initializeTokenCC();
        headers.add("Authorization", "Bearer "+getJwt());
        HttpEntity<Map<String, Object>> httpEntity = new HttpEntity<>(headers);

        List<ApplicationDto> result = new ArrayList<>();
        List<Task> tasks;
        for(ApplicationDto applicationDto : applicationsDto)
        {
            String requestUrl = applicationDto.getLinkBack()+"/request/detail/"+email;
            try
            {
                Task[] response = restTemplate.exchange(
                        requestUrl,
                        HttpMethod.GET,
                        httpEntity,
                        Task[].class
                ).getBody();
                tasks = Arrays.asList(response);
            }
            catch(Exception e){
                tasks = new ArrayList<>();
            }
                 applicationDto.setTasks(tasks);
                 result.add(applicationDto);
        }
        return result;
    }
