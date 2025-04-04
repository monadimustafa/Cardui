Rimport static org.mockito.Mockito.*;
import static org.junit.Assert.*;

import ma.sg.its.rpaportal.services.OrchestratorService;
import ma.sg.its.rpaportal.services.impl.FileWatcher;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.junit.Before;
import org.junit.Test;
import org.mockito.*;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class FileWatcherTest {

    @Mock
    private OrchestratorService orchestratorService; // Simuler le service Orchestrator

    @InjectMocks
    private FileWatcher fileWatcher; // Instancier FileWatcher avec les mocks

    @Before
    public void setUp() {
        // Initialiser les mocks
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles() throws IOException {
        // Créer un fichier temporaire pour le test
        File tempFile = File.createTempFile("test-file", ".xlsx");
        tempFile.deleteOnExit();

        // Simuler le répertoire contenant ce fichier
        String directoryPath = tempFile.getParent();
        File directory = new File(directoryPath);
        
        // Simuler la méthode listFiles() pour qu'elle retourne notre fichier temporaire
        when(directory.listFiles()).thenReturn(new File[]{tempFile});

        // Simuler la méthode takeDecision() pour retourner true
        when(orchestratorService.takeDecision(anyInt(), anyString())).thenReturn(true);

        // Exécuter la méthode à tester
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Vérifier que le fichier a été déplacé
        Path targetPath = Paths.get(directoryPath, "archive", tempFile.getName());
        assertTrue(Files.exists(targetPath));

        // Vérifier que la décision a bien été prise (par exemple, vérifier l'appel à takeDecision)
        verify(orchestratorService).takeDecision(anyInt(), anyString());

        // Nettoyer les ressources
        Files.delete(targetPath); // Nettoyage du fichier déplacé
    }

    @Test
    public void testMoveFile() throws IOException {
        // Créer un fichier temporaire
        File tempFile = File.createTempFile("test-file", ".txt");
        tempFile.deleteOnExit();
        
        // Simuler le chemin cible
        String targetDirectory = tempFile.getParent() + "/archive";

        // Appeler la méthode moveFile
        fileWatcher.moveFile(tempFile.getAbsolutePath(), targetDirectory);

        // Vérifier si le fichier existe à l'emplacement cible
        Path targetPath = Paths.get(targetDirectory, tempFile.getName());
        assertTrue(Files.exists(targetPath));

        // Nettoyer les ressources
        Files.delete(targetPath);
    }

    @Test
    public void testMoveFileException() {
        // Tester la gestion des exceptions lors du déplacement de fichier
        try {
            fileWatcher.moveFile("invalid/path/to/file.txt", "invalid/target");
            fail("Expected IOException");
        } catch (IOException e) {
            // Vérifier que l'exception est correctement gérée
            assertNotNull(e);
        }
    }
}
