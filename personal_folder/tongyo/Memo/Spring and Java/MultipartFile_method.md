# MultipartFile Method

|메소드명|설명|
|----|----|
|String getName()|파라미터의 이름 \<input> 태그의 이름|
|String getOriginalFileName()|업로드되는 파일의 이름|
|boolean isEmpty()|파일이 존재하지 않는 경우 true|
|long getSize()|업로드되는 파일의 크기|
|byte[] getByte()|byte[]로 파일 데이터 변환|
|InputStream getInputStream()|파일데이터와 연결된 InputStream 반환|
|transferTo(File file)|파일의 저장|