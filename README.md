import org.junit.Before;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.junit.runner.RunWith;
import org.mockito.junit.MockitoJUnitRunner;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;

import static org.mockito.Mockito.*;
import static org.junit.Assert.*;

@RunWith(MockitoJUnitRunner.class)
public class FileWatcherTest {

    @Mock
    private Service service; // Assuming 'service' is a class with a takeDecision method

    @Mock
    private Logger log;

    @InjectMocks
    private FileWatcher fileWatcher; // Assuming 'FileWatcher' is the class containing your method

    private String tempDirPath;

    @Before
    public void setUp() throws IOException {
        MockitoAnnotations.openMocks(this);
        tempDirPath = System.getProperty("java.io.tmpdir") + "/testFiles";
        File tempDir = new File(tempDirPath);
        if (!tempDir.exists()) {
            tempDir.mkdirs();
        }
        fileWatcher.directories = Arrays.asList(tempDirPath); // Ensure the directory list is initialized
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_ATD() throws IOException, InvalidFormatException {
        // Arrange
        File atdDir = new File(tempDirPath + "/ATD");
        atdDir.mkdirs();
        File testFile = new File(atdDir, "test.xlsx");
        XSSFWorkbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Nouvelle ATD");
        sheet.createRow(0);
        sheet.createRow(1);
        workbook.write(testFile);
        workbook.close();

        when(service.takeDecision(2, "ATD")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, times(1)).takeDecision(2, "ATD");
        assertTrue(new File(atdDir.getParentFile() + "/archive/test.xlsx").exists());
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_CC() throws IOException, InvalidFormatException {
        // Arrange
        File ccDir = new File(tempDirPath + "/CC");
        ccDir.mkdirs();
        File testFile = new File(ccDir, "test.csv");
        String csvContent = "header1;header2\nvalue1;value2\nvalue3;value4";
        java.nio.file.Files.write(testFile.toPath(), csvContent.getBytes());

        when(service.takeDecision(2, "CC")).thenReturn(true);
        when(fileWatcher.headersCC()).thenReturn(new String[]{"header1", "header2"});

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, times(1)).takeDecision(2, "CC");
        assertTrue(new File(ccDir.getParentFile() + "/archive/test.csv").exists());
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_Other() throws IOException, InvalidFormatException {
        // Arrange
        File otherDir = new File(tempDirPath + "/Other");
        otherDir.mkdirs();
        File testFile = new File(otherDir, "test.xlsx");
        XSSFWorkbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Sheet1");
        sheet.createRow(0);
        sheet.createRow(1);
        sheet.createRow(2);
        workbook.write(testFile);
        workbook.close();

        when(service.takeDecision(3, "Other")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, times(1)).takeDecision(3, "Other");
        assertTrue(new File(otherDir.getParentFile() + "/archive/test.xlsx").exists());
    }

    @Test
    public void testCheckInputDirectoriesForNewFiles_NoFiles() throws IOException, InvalidFormatException {
        // Arrange
        // No files are created in the test directory

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, never()).takeDecision(anyInt(), anyString());
    }

    // Assuming you have a method headersCC() in your FileWatcher class.
    public String[] headersCC() {
        return new String[]{"header1", "header2"};
    }

    public static class Service {
        public boolean takeDecision(int fileLength, String parentName) {
            return true;
        }
    }

    public static class FileWatcher{
        public List<String> directories;
        private Service service;
        private Logger log;

        public FileWatcher(Service service, Logger log) {
            this.service = service;
            this.log = log;
        }

        public FileWatcher() {

        }

        @Scheduled(cron = "${rpa-portal.seed.orchestrator.cron}")
        public void checkInputDirectoriesForNewFiles() throws IOException, InvalidFormatException {
            log.info("[FileWatcher - checkInputDirectoriesForNewFiles] start");
            for (String directory : directories) {
                File[] files = new File(directory).listFiles();
                if (files != null) {
                    for (File file : files) {
                        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] Found A new File: {}", file.getAbsolutePath());
                        Workbook workbook ;
                        int fileLenght=0;
                        Sheet sheet;
                        if(file.getParentFile().getParentFile().getName().equals("ATD"))
                        {
                            workbook = new XSSFWorkbook(file);
                            sheet = workbook.getSheet("Nouvelle ATD");
                            fileLenght = sheet.getLastRowNum()+1;
                        }
                        else if(file.getParentFile().getParentFile().getName().equals("CC"))
                        {
                            FileReader fileReader = new FileReader(file.getAbsolutePath());
                            CSVParser parser = new CSVParser(fileReader, CSVFormat.RFC4180
                                    .withHeader(this.headersCC())
                                    .withIgnoreEmptyLines()
                                    .withDelimiter(';')
                                    .withFirstRecordAsHeader());
                            fileLenght=parser.getRecords().size();
                            fileReader.close();
                        }
                        else
                        {
                            workbook = new XSSFWorkbook(file);
                            sheet = workbook.getSheetAt(0);
                            fileLenght = sheet.getLastRowNum()+1;
                        }
                        boolean dicision = service.takeDecision(fileLenght, file.getParentFile().getParentFile().getName());
                        if(dicision)
                            moveFile(file.getAbsolutePath(), file.getParentFile().getParentFile() + "/archive");
                    }
                }
            }
            log.info("[FileWatcher - checkInputDirectoriesForNewFiles] End");
        }

        public String[] headersCC(){
            return new String[]{"header1","header2"};
        }

        public void moveFile(String source, String destination) throws IOException{
            File sourceFile = new File(source);
            File destDir = new File(destination);
            destDir.mkdirs();
            File destFile = new File(destination, sourceFile.getName());
            sourceFile.renameTo(destFile);
        }
    }
}
