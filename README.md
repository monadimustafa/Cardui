Dimport static org.mockito.Mockito.*;
import static org.junit.Assert.*;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.commons.csv.*;
import org.junit.Before;
import org.junit.Test;
import org.mockito.*;

import java.io.*;
import java.nio.file.*;
import java.util.*;

public class FileWatcherTest {

    @Mock
    private FileWatcherService service;

    @InjectMocks
    private FileWatcher fileWatcher;

    @Mock
    private File file;

    @Mock
    private XSSFWorkbook workbook;

    @Mock
    private CSVParser csvParser;

    @Mock
    private Sheet sheet;

    private List<String> directories;
    private String testDirectory = "testDir";

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        directories = new ArrayList<>();
        directories.add(testDirectory);
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_XSSFWorkbook() throws Exception {
        // Arrange
        String fileName = "testFile.xlsx";
        when(file.getName()).thenReturn(fileName);
        when(file.isFile()).thenReturn(true);
        when(file.getParentFile()).thenReturn(new File("ATD"));
        when(file.getAbsolutePath()).thenReturn(testDirectory + "/" + fileName);

        XSSFWorkbook mockWorkbook = mock(XSSFWorkbook.class);
        Sheet mockSheet = mock(Sheet.class);
        when(mockWorkbook.getSheet("Nouvelle ATD")).thenReturn(mockSheet);
        when(mockSheet.getLastRowNum()).thenReturn(100); // mock the row count

        when(service.takeDecision(100, "ATD")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service).takeDecision(100, "ATD");
        verify(file, times(1)).getAbsolutePath(); // Ensure method is called once
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_CSVFile() throws Exception {
        // Arrange
        String fileName = "testFile.csv";
        when(file.getName()).thenReturn(fileName);
        when(file.isFile()).thenReturn(true);
        when(file.getParentFile()).thenReturn(new File("CC"));
        when(file.getAbsolutePath()).thenReturn(testDirectory + "/" + fileName);

        CSVParser mockParser = mock(CSVParser.class);
        List<CSVRecord> records = Arrays.asList(mock(CSVRecord.class), mock(CSVRecord.class));
        when(mockParser.getRecords()).thenReturn(records);

        when(service.takeDecision(records.size(), "CC")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service).takeDecision(records.size(), "CC");
        verify(file, times(1)).getAbsolutePath(); // Ensure method is called once
    }

    @Test
    public void testMoveFile() throws IOException {
        // Arrange
        Path sourcePath = Paths.get("sourcePath");
        Path targetPath = Paths.get("targetPath");
        when(file.getAbsolutePath()).thenReturn(sourcePath.toString());

        // Act
        fileWatcher.moveFile(sourcePath.toString(), "targetPath");

        // Assert
        verify(service, times(1)).moveFile(anyString(), anyString()); // Verify move call
    }
}
