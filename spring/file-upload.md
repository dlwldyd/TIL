# 파일 업로드
## multipart/form-data
<img src="./../img/multipart-form-data.png">

* 파일을 업로드할 때는 content-type을 multipart/form-data를 사용한다.
* multipart/form-data를 통해 다른 종류의 여러 파일과 폼의 내용 함께 전송할 수 있다. 각각의 폼은 boundary의 문자열로 나누어져있다.
## 클라이언트가 서버에 파일 업로드
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
</head>
<body>
<div>
    <form th:action method="post" enctype="multipart/form-data">
        <ul>
            <li>이름 <input type="text" name="name"></li>
            <li>파일<input type="file" name="file" ></li>
        </ul>
        <input type="submit"/>
    </form>
</div>
</body>
</html>
```
```java
@Controller
@RequestMapping("/upload")
public class UploadController {
    //application.properties의 file.dir값을 받는다.(파일을 저장할 기본 경로)
    @Value("${file.dir}")
    private String fileDir;

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFile(@RequestParam String name, @RequestParam MultipartFile file) throws IOException {
        if (!file.isEmpty()) {
            String fullPath = fileDir + file.getOriginalFilename();
            file.transferTo(new File(fullPath));
        }

        return "upload-form";
    }
}
```
* @RequestParam MultipartFile file : 업로드된 파일이 바인딩 된다.
* file.getOriginalFilename() : 업로드 파일명을 반환한다.
* file.transferTo(new File(fullPath)) : 파일을 지정한 경로에 저장한다.
## 서버에서 클라이언트로
```java
@Data
@AllArgsConstructor
public class UploadFile {
    //클라이언트에서 업로드한 파일명
    private String uploadFileName;  
    //서버측에서 저장할 때의 파일명  
    private String storeFileName;
}
```
* 클라이언트에서 업로드할 때의 파일명을 그대로 쓰면 파일명이 중복될 수 있기 때문에 클라이언트에서 업로드할 때의 파일명과 서버에서 저장할 때의 파일명을 따로 관리해주어야 한다.
* 뷰를 렌더링할 때 지정된 파일경로에서 서버측에서 저장한 파일명으로 파일을 찾고, 클라이언트가 해당 파일을 다운로드 할때 클라이언트에서 업로드했을 때의 파일명으로 파일을 넘긴다.
```java
@Data
public class Item {
    private Long id;
    private String itemName;
    //클라이언트가 다운로드할 파일
    private UploadFile attachFile;
    //이미지 파일
    private List<UploadFile> imageFiles;

    public Item(String itemName, UploadFile attachFile, List<UploadFile> imageFiles) {
        this.itemName = itemName;
        this.attachFile = attachFile;
        this.imageFiles = imageFiles;
    }
}
```
```java
@Repository
public class ItemRepository {

    private final Map<Long, Item> store = new HashMap<>();
    private long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }
}
```
* 데이터베이스에는 파일이름이나 경로정도만 저장하고 실제 파일은 파일 저장소에 따로 저장한다.
```java
@Data
public class ItemForm {
    private Long itemId;
    private String itemName;
    private MultipartFile attachFile;
    private List<MultipartFile> imageFiles;
}
```
```java
@Component
public class FileStore {

    @Value("${file.dir}")
    public String fileDir;

    public String getFullPath(String fileName) {
        return fileDir + fileName;
    }

    //여러개의 파일을 업로드할 때 사용되는 함수
    public List<UploadFile> storeFiles(List<MultipartFile> multipartFiles) throws IOException {
        List<UploadFile> storeFileResult = new ArrayList<>();
        for (MultipartFile multipartFile : multipartFiles) {
            storeFileResult.add(storeFile(multipartFile));
        }
        return storeFileResult;
    }

    //데이터베이스와 파일저장소에 저장하는 함수
    public UploadFile storeFile(MultipartFile multipartFile) throws IOException {
        if (multipartFile.isEmpty()) {
            return null;
        }

        String originalFilename = multipartFile.getOriginalFilename();

        String storeFileName = createStoreFileName(originalFilename);

        multipartFile.transferTo(new File(getFullPath(storeFileName)));

        return new UploadFile(originalFilename, storeFileName);
    }

    //파일이름을 UUID를 이용해서 중복되지 않도록 저장한다.
    private String createStoreFileName(String originalFilename) {
        String uuid = UUID.randomUUID().toString();
        String ext = extractExt(originalFilename);

        return uuid + ext;
    }
    
    //파일의 확장자만 뽑아내는 함수
    private String extractExt(String originalFilename) {
        int pos = originalFilename.lastIndexOf(".");
        return originalFilename.substring(pos);
    }
}
```
```html
<!--item-form.html-->
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
</head>
<body>
<div>
    <form th:action method="post" enctype="multipart/form-data">
        <ul>
            <li>상품명 <input type="text" name="itemName"></li>
            <li>첨부파일<input type="file" name="attachFile" ></li>
            <li>이미지 파일들<input type="file" multiple="multiple" name="imageFiles" ></li>
        </ul>
        <input type="submit"/>
    </form>
</div>
</body>
</html>
```
```html
<!--item-view.html-->
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
</head>
<body>
<div>
    상품명: <span th:text="${item.itemName}">상품명</span><br/>
    첨부파일: <a th:if="${item.attachFile}" th:href="|/attach/${item.id}|"
             th:text="${item.getAttachFile().getUploadFileName()}" /><br/>
    <img th:each="imageFile : ${item.imageFiles}" th:src="|/images/${imageFile.getStoreFileName()}|" width="300" height="300"/>
</div>
</body>
</html>
```
```java
@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemRepository itemRepository;
    private final FileStore fileStore;

    //파일을 첨부하는 페이지
    @GetMapping("/items/new")
    public String newItem(@ModelAttribute ItemForm form) {
        return "item-form";
    }

    //파일을 첨부하고 submit 했을 때
    @PostMapping("/items/new")
    public String saveItem(@ModelAttribute ItemForm form, RedirectAttributes redirectAttributes) throws IOException {
        UploadFile attachFile = fileStore.storeFile(form.getAttachFile());
        List<UploadFile> storeImageFiles = fileStore.storeFiles(form.getImageFiles());

        Item item = new Item(form.getItemName(), attachFile, storeImageFiles);
        itemRepository.save(item);

        redirectAttributes.addAttribute("itemId", item.getId());

        return "redirect:/items/{itemId}";
    }

    //서버측으로부터 이미지와 파일을 다운로드 받는 페이지
    @GetMapping("/items/{id}")
    public String items(@PathVariable Long id, Model model) {
        Item item = itemRepository.findById(id);
        model.addAttribute("item", item);
        return "item-view";
    }

    //이미지를 다운로드 받을 때
    @ResponseBody
    @GetMapping("images/{filename}")
    public Resource downloadImage(@PathVariable String filename) throws MalformedURLException {
        return new UrlResource("file:" + fileStore.getFullPath(filename));
    }

    //파일을 다운로드 받을 때
    @GetMapping("/attach/{itemId}")
    public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
        Item item = itemRepository.findById(itemId);
        String storeFileName = item.getAttachFile().getStoreFileName();
        String uploadFileName = item.getAttachFile().getUploadFileName();

        UrlResource resource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));

        log.info("uploadFileName={}", uploadFileName);

        //파일이름이 한글이면 깨지기 때문에 utf-8로 인코딩 해주어야한다.
        String encodedUploadFile = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);
        //첨부파일임을 나타내는 헤더값을 넣어야 다운로드할 수 있다. 만약 이 헤더값을 넣지 않으면 다운로드가 아니라 브라우저에 그대로 노출된다.
        String contentDisposition = "attachment; filename=\"" + encodedUploadFile + "\"";

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
                .body(resource);
    }
}
```
* 파일이 컨트롤러에서 Argument Resolver에 의해 바인딩될 때에는 MultipartFile형식으로 바인딩된다. 하지만 뷰를 렌더링할 때에는 파일의 경로(String)를 이용해 파일을 찾고, 업로드될 때의 파일이름과 저장소에 저장할 때의 파일이름을 다르게 사용하기 때문에 이를 위한 객체를 따로 만들어 줘야한다.
* @GetMapping("/images/{filename}") : \<img> 태그로 이미지를 조회할 때 사용한다. UrlResource에 "file:(파일의 경로)"를 파라미터로 넣어 객체를 생성해 이미지 파일을 읽어서 @ResponseBody 로 이미지 바이너리를 반환한다.(Resource 객체를 반환해야한다. UrlResouce는 Resource의 자식 클래스이다)
* 클라이언트가 파일을 브라우저에서 읽도록 하는게 아니라 다운로드를 하게 만드려면 ContentDisposition 헤더값을 attachment; filename="(파일의 경로)"로 세팅해야한다.(ResponseEntity 사용)