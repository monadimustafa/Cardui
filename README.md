import org.springframework.http.*;
import org.springframework.web.client.HttpStatusCodeException;
import org.springframework.web.client.ResourceAccessException;
import org.springframework.web.client.RestTemplate;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;

public List<ApplicationDto> getRequests(List<ApplicationDto> applicationsDto, String email) {
    RestTemplate restTemplate = new RestTemplate();
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    initializeTokenCC();
    headers.add("Authorization", "Bearer " + getJwt());
    HttpEntity<Map<String, Object>> httpEntity = new HttpEntity<>(headers);

    List<ApplicationDto> result = new ArrayList<>();
    for (ApplicationDto applicationDto : applicationsDto) {
        String requestUrl = applicationDto.getLinkBack() + "/request/detail/" + email;
        List<Task> tasks;
        try {
            ResponseEntity<Task[]> response = restTemplate.exchange(
                    requestUrl,
                    HttpMethod.GET,
                    httpEntity,
                    Task[].class
            );

            if (response.getStatusCode() == HttpStatus.OK) {
                Task[] responseBody = response.getBody();
                tasks = responseBody != null ? Arrays.asList(responseBody) : new ArrayList<>();
            } else {
                tasks = new ArrayList<>();
            }
        } catch (HttpStatusCodeException e) {
            // Handle specific HTTP status codes
            System.err.println("HTTP error: " + e.getStatusCode());
            tasks = new ArrayList<>();
        } catch (ResourceAccessException e) {
            // Handle resource access errors (e.g., timeouts)
            System.err.println("Resource access error: " + e.getMessage());
            tasks = new ArrayList<>();
        } catch (Exception e) {
            // Handle other exceptions
            System.err.println("An error occurred: " + e.getMessage());
            tasks = new ArrayList<>();
        }

        // Check if the list of tasks is empty and handle accordingly
        if (tasks.isEmpty()) {
            System.err.println("No tasks found for: " + requestUrl);
        }

        applicationDto.setTasks(tasks);
        result.add(applicationDto);
    }
    return result;
}


fetchApplications() {
    this.applicationService.getRequests(this.email).subscribe(
      (result: ApplicationDto[]) => {
        this.applications = result.map(application => {
          if (application.tasks && application.tasks.length > 0) {
            application['status'] = 'success'; // Tâches reçues
          } else {
            application['status'] = 'no-tasks'; // Pas de tâches
          }
          return application;
        });
      },
      error => {
        this.applications = this.applications.map(application => {
          application['status'] = 'error'; // Erreur
          return application;
        });
        this.handleError(error);
      }
    );
  }

  private handleError(error: any) {
    console.error('Error fetching applications:', error);
    // Vous pouvez ajouter des messages utilisateur ou d'autres actions ici
  }
