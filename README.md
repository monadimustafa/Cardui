@Scheduled(cron = "${rpa-portal.seed.orchestrator.cron}")
    public void checkInputDirectoriesForNewFiles() throws IOException, InvalidFormatException {
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] start");
        for (String directory : directories) {
            File[] files = new File(directory).listFiles();
            if (files != null) {
                for (File file : files) {
                    if(file.isFile()){
                        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] Found A new File: {}", file.getAbsolutePath());
                        Workbook workbook ;
                        int fileLenght=0;
                        Sheet sheet;
                        if(file.getParentFile().getParentFile().getName().equals("ATD")) {
                            try (XSSFWorkbook workbookResource = new XSSFWorkbook(file)) {
                                workbook = workbookResource;
                                sheet = workbook.getSheet("Nouvelle ATD");
                                fileLenght = sheet.getLastRowNum();
                            }
                             catch (IOException e) {
                                    e.printStackTrace(); // Handle exception appropriately
                                    log.error("Error processing workbook for file {}: {}", file.getAbsolutePath(), e.getMessage());
                                }
                        }
                        else if(file.getParentFile().getParentFile().getName().equals("CC"))
                        {
                            try(FileReader fileReader = new FileReader(file.getAbsolutePath())){
                                CSVParser parser = new CSVParser(fileReader, CSVFormat.RFC4180
                                        .withHeader(this.headersCC())
                                        .withIgnoreEmptyLines()
                                        .withDelimiter(';')
                                        .withFirstRecordAsHeader());
                                fileLenght=parser.getRecords().size();
                            }
                            catch (IOException e) {
                                e.printStackTrace(); // Handle exception appropriately
                                log.error("Error processing CSV for file {}: {}", file.getAbsolutePath(), e.getMessage());
                            }
                        }
                        else
                        {
                            workbook = new XSSFWorkbook(file);
                            sheet = workbook.getSheetAt(0);
                            fileLenght = sheet.getLastRowNum();
                        }
                        boolean dicision = service.takeDecision(fileLenght, file.getParentFile().getParentFile().getName());
                        if(dicision)
                            moveFile(file.getAbsolutePath(), file.getParentFile().getParentFile() + "/archive");
                    }
                    else{
                        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] not found");
                    }
                    }
            }
        }
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] End");
    }
    public void moveFile(String filePath, String target) throws IOException {
        Path sourcePath = Paths.get(filePath);
        Path targetPath = Paths.get(target);

        try {
            // Move the file
            Files.move(sourcePath, targetPath.resolve(sourcePath.getFileName()), StandardCopyOption.REPLACE_EXISTING);

            log.info("File with path {} moved successfully to {}", filePath, target);
        } catch (IOException e) {
            log.error("Error moving the file {} to {}", filePath, target, e);
            throw e; // Rethrow the IOException
        }
    }
