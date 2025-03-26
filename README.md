[ERROR]   FileWatcherTest.testCheckInputDirectoriesForNewFiles_ATD:72 » IllegalArgument ...
[ERROR]   FileWatcherTest.testCheckInputDirectoriesForNewFiles_CC:88 MissingMethodInvocation
[ERROR]   FileWatcherTest.testCheckInputDirectoriesForNewFiles_NoFiles:128 » IllegalArgument
[ERROR]   FileWatcherTest.testCheckInputDirectoriesForNewFiles_Other:116 » IllegalArgument
[ERROR]   PortalSeedServiceImplTest.testProcessBasketFile_ExceptionHandling:451 NotAMock
[ERROR]   PortalSeedServiceImplTest.testProcessBasketFile_ForCCDirectory:430 » NullPointer
[ERROR]   PortalSeedServiceImplTest.testProcessBasketFile_ForNonCCDirectory:411 NotAMock

package ma.sg.its.rpaportal.services.impl;

import com.fasterxml.jackson.databind.exc.InvalidFormatException;
import ma.sg.its.rpaportal.services.OrchestratorService;
import org.apache.catalina.Container;
import org.apache.juli.logging.Log;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.MockitoJUnitRunner;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;

import static org.apache.commons.io.FileUtils.moveFile;
import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class FileWatcherTest {
    @Mock
    private OrchestratorService service;
    @Mock
    private Logger log;

    @InjectMocks
    private FileWatcher fileWatcher;
    private String tempDirPath;

    @Before
    public void setUp() throws IOException {
        MockitoAnnotations.initMocks(this);
        tempDirPath = System.getProperty("java.io.tmpdir") + "/testFiles";
        File tempDir = new File(tempDirPath);
        if (!tempDir.exists()) {
            tempDir.mkdirs();
        }
        fileWatcher.directories = Arrays.asList(tempDirPath); // Ensure the directory list is initialized
    }
    @Test
    public void testCheckInputDirectoriesForNewFiles_ATD() throws IOException, InvalidFormatException, org.apache.poi.openxml4j.exceptions.InvalidFormatException {
        // Arrange
        File atdDir = new File(tempDirPath + "/ATD");
        atdDir.mkdirs();
        File testFile = new File(atdDir, "test.xlsx");
        XSSFWorkbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Nouvelle ATD");
        sheet.createRow(0);
        sheet.createRow(1);
        try(FileOutputStream fileOutputStream = new FileOutputStream(testFile)){
            workbook.write(fileOutputStream);
        }

        workbook.close();
        when(service.takeDecision(2, "ATD")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, times(1)).takeDecision(2, "ATD");
        assertTrue(new File(atdDir.getParentFile() + "/archive/test.xlsx").exists());
    }
    @Test
    public void testCheckInputDirectoriesForNewFiles_CC() throws IOException, InvalidFormatException, org.apache.poi.openxml4j.exceptions.InvalidFormatException {
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
    public void testCheckInputDirectoriesForNewFiles_Other() throws IOException, InvalidFormatException, org.apache.poi.openxml4j.exceptions.InvalidFormatException {
        // Arrange
        File otherDir = new File(tempDirPath + "/Other");
        otherDir.mkdirs();
        File testFile = new File(otherDir, "test.xlsx");
        XSSFWorkbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Sheet1");
        sheet.createRow(0);
        sheet.createRow(1);
        sheet.createRow(2);
        try(FileOutputStream fileOutputStream = new FileOutputStream(testFile)){
            workbook.write(fileOutputStream);
        }
        workbook.close();

        when(service.takeDecision(3, "Other")).thenReturn(true);

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, times(1)).takeDecision(3, "Other");
        assertTrue(new File(otherDir.getParentFile() + "/archive/test.xlsx").exists());
    }
    @Test
    public void testCheckInputDirectoriesForNewFiles_NoFiles() throws IOException, InvalidFormatException, org.apache.poi.openxml4j.exceptions.InvalidFormatException {
        // Arrange
        // No files are created in the test directory

        // Act
        fileWatcher.checkInputDirectoriesForNewFiles();

        // Assert
        verify(service, never()).takeDecision(anyInt(), anyString());
    }

    @Test
    public void testMoveFile() throws IOException {
        // Arrange
        Path tempSource = Files.createTempFile("testFile", ".txt");
        Path tempTargetDir = Files.createTempDirectory("testArchive");
        String targetPath = tempTargetDir.toString();
        FileWatcher watcher = new FileWatcher(service);

        // Act
        watcher.moveFile(tempSource.toString(), targetPath);

        // Assert
        Path movedFile = tempTargetDir.resolve(tempSource.getFileName());
        assertTrue(Files.exists(movedFile));
        Files.deleteIfExists(movedFile);
        Files.deleteIfExists(tempTargetDir);
    }

}
