public ResponseEntity<List<ApplicationDto>> getPortalRequestForRpa(
            @RequestHeader(name = "X-USER", required = false) String userEmail) throws Exception {
        log.info("[Portal - getPortalRequestForRpa] Start resource Analytics Ressource");
        PortalUser portalUser = authorizationService.authorize(userEmail, Action.AUDIT_PILOTAGE);
        List<ApplicationDto> applicationDtos = portalUser.getApplications().stream()
                .map(userMapper::fromApplicationToDTO)
                .collect(Collectors.toList());

        List<ApplicationDto> response = restTemplateConsumer.getRequests(applicationDtos, userEmail);

        calculateCountTask(response);

        return ResponseEntity.ok(response);
    }

    public void calculateCountTask(List<ApplicationDto> response) {
        for (ApplicationDto applicationDto : response) {
            int count = 0;
            for (PortalRequestDTOExtern portalRequestDTOExtern : applicationDto.getPortalRequestDTOExternList()) {
                count += portalRequestDTOExtern.getCount();
            }
            applicationDto.setCountTask(count);
        }
    }



    import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;

public ResponseEntity<List<ApplicationDto>> getPortalRequestForRpa(
        @RequestHeader(name = "X-USER", required = false) String userEmail) {
    try {
        log.info("[Portal - getPortalRequestForRpa] Start Resource Analytics Ressource");

        // Authorize the user based on the provided email
        PortalUser portalUser = autorisationService.authorize(userEmail, Action.AUDIT_PILOTAGE);

        // Map applications to DTOs and collect them into a list
        List<ApplicationDto> applicationDtos = portalUser.getApplications().stream()
                .map(userMapper::fromApplicationToDTO)
                .collect(Collectors.toList());

        // Get portal requests for the applications
        List<ApplicationDto> response = restTemplateConsumer.getRequests(applicationDtos, userEmail);

        // Calculate count of tasks for each application DTO in the response
        calculateCountTask(response);

        // Return the response entity with the list of application DTOs
        return ResponseEntity.ok(response);
    } catch (Exception e) {
        log.error("Error occurred while processing portal request for RPA", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}

public void calculateCountTask(List<ApplicationDto> response) {
    for (ApplicationDto applicationDto : response) {
        int count = 0;
        for (PortalRequestDTOExtern portalRequestDTOExtern : applicationDto.getPortalRequestDTOExternList()) {
            count += portalRequestDTOExtern.getCount();
        }
        applicationDto.setCountTask(count);
    }
}


import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.*;

public class PortalControllerTest {

    @Mock
    AutorisationService autorisationService;
    @Mock
    RestTemplateConsumer restTemplateConsumer;
    @Mock
    UserMapper userMapper;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetPortalRequestForRpa_ExceptionHandling() {
        PortalController controller = new PortalController(autorisationService, restTemplateConsumer, userMapper);
        String userEmail = "test@example.com";

        // Mocking exception scenario for autorisationService.authorize
        when(autorisationService.authorize(userEmail, Action.AUDIT_PILOTAGE)).thenThrow(new Exception("Authorization failed"));

        ResponseEntity<List<ApplicationDto>> responseEntity = controller.getPortalRequestForRpa(userEmail);

        assertEquals(HttpStatus.INTERNAL_SERVER_ERROR, responseEntity.getStatusCode());
        assertNull(responseEntity.getBody());
    }
}

