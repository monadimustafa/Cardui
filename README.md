 public static void writeFileToDirectory(PortalRequest portalRequest) {
        String pCode = portalRequest.getRequestNumber().split("-")[0]; // Récupérer 'CC'
        String remainingPart = portalRequest.getRequestNumber().substring(pCode.length() + 1); // Récupérer le reste du numéro

        String prefix = portalRequest.getOrigin().equals("INPUT") ? "M" : "A"; // Définir 'M' ou 'A'

        String fileName = pCode + "-" + prefix + "-" + remainingPart + portalRequest.getRpAttachment().getExtension();


        String base64Content = portalRequest.getRpAttachment().getLoadedFileBase64();
        String processCode = portalRequest.getRequestNumber().substring(0, portalRequest.getRequestNumber().indexOf("-"));
        String directoryPath = "data/RPA/" + processCode + "/input";
        String targetFolderPath = File.separator;

        File directory = new File(targetFolderPath + directoryPath);
        if (!directory.exists() && !directory.mkdirs()) {
            throw new RpaPortalException("Failed to create directory: " + directory.getAbsolutePath());
        }

        // Create the original file
        File file = new File(targetFolderPath + directoryPath, fileName);

        try (FileOutputStream outputStream = new FileOutputStream(file); Writer writer = new OutputStreamWriter(outputStream, StandardCharsets.UTF_8) ) {
            // Decode the Base64 content

            byte[] decodedContent = Base64.getDecoder().decode(base64Content);
            String content = new String(decodedContent,StandardCharsets.UTF_8);
            content = content.replaceAll("(?<=,|^)(\\d{15,})(?=,|$)", "=\"$1\"");

            writer.write("\uFEFF");
            writer.write(content);
            log.info("File written successfully: {}", file.getAbsolutePath());
        } catch (IOException e) {
            throw new RpaPortalException(e.getMessage());
        }
    }
