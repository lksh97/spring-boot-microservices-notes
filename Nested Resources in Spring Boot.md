# Working with Nested Resources in Spring Boot

## 1. Introduction to Nested Resources

### Kya hai Nested Resources?

Nested resources ek REST API design pattern hai jisme ek resource dusre resource ke andar ya uske saath relationship mein hota hai. For example, ek Course ke andar Videos, ek User ke andar Posts, ya ek Product ke andar Reviews.

Nested resources ka use karne se aapke API endpoints more intuitive and organized ho jaate hain, kyunki ye real-world relationships ko reflect karte hain.

### RESTful API mein Nested Resources kyu important hai?

1. **Logical Hierarchy** - Real-world objects ke beech natural parent-child relationships ko represent karta hai
2. **Better Organization** - Related resources ko group karne se API more intuitive ban jati hai
3. **Improved Developer Experience** - Developers jaldi samajh sakte hain ki resources kaise related hain

### Resource Relationships ke Types

1. **One-to-Many**: Ek parent resource ke multiple child resources (e.g., ek Course ke multiple Videos)
2. **Many-to-Many**: Resources ke beech many-to-many relationship (e.g., Courses aur Categories)
3. **One-to-One**: Ek resource ka ek related resource (e.g., User aur UserProfile)

## 2. URL Structure for Nested Resources

### Standard URL Patterns

Nested resources ke liye URL patterns generally is tarah hote hain:

```
/parents/{parentId}/children
/parents/{parentId}/children/{childId}
```

Example:

```
/courses/{courseId}/videos           # Get all videos for a course
/courses/{courseId}/videos/{videoId} # Get specific video of a course
```

### Examples

| Operation | HTTP Method | URL | Description |
|-----------|-------------|-----|-------------|
| List all courses | GET | `/courses` | Retrieve all courses |
| Get one course | GET | `/courses/{courseId}` | Retrieve a specific course |
| List all videos of a course | GET | `/courses/{courseId}/videos` | Retrieve all videos for a specific course |
| Get one video of a course | GET | `/courses/{courseId}/videos/{videoId}` | Retrieve a specific video for a specific course |
| Create a video for a course | POST | `/courses/{courseId}/videos` | Create a new video for a course |
| Update a video of a course | PUT | `/courses/{courseId}/videos/{videoId}` | Update a specific video of a course |
| Delete a video from a course | DELETE | `/courses/{courseId}/videos/{videoId}` | Delete a specific video from a course |

## 3. Implementing Nested Resources in Spring Boot

### Controller-Level Implementation

Nested resources ko implement karne ke liye controller mein `@RequestMapping` ka use karein:

```java
@RestController
@RequestMapping("/api/v1/courses/{courseId}/videos")
public class CourseVideoController {
    
    private final VideoService videoService;
    
    public CourseVideoController(VideoService videoService) {
        this.videoService = videoService;
    }
    
    @GetMapping
    public ResponseEntity<List<VideoDto>> getAllVideosForCourse(@PathVariable String courseId) {
        return ResponseEntity.ok(videoService.getVideosByCourseId(courseId));
    }
    
    @GetMapping("/{videoId}")
    public ResponseEntity<VideoDto> getVideoInCourse(
            @PathVariable String courseId,
            @PathVariable String videoId) {
        return ResponseEntity.ok(videoService.getVideoInCourse(courseId, videoId));
    }
    
    @PostMapping
    public ResponseEntity<VideoDto> createVideoForCourse(
            @PathVariable String courseId,
            @RequestBody VideoDto videoDto) {
        // Set the course ID in the video DTO
        videoDto.setCourseId(courseId);
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(videoService.createVideo(videoDto));
    }
    
    @PutMapping("/{videoId}")
    public ResponseEntity<VideoDto> updateVideoInCourse(
            @PathVariable String courseId,
            @PathVariable String videoId,
            @RequestBody VideoDto videoDto) {
        // Ensure proper relationship
        videoDto.setVideoId(videoId);
        videoDto.setCourseId(courseId);
        return ResponseEntity.ok(videoService.updateVideo(videoDto));
    }
    
    @DeleteMapping("/{videoId}")
    public ResponseEntity<Void> deleteVideoFromCourse(
            @PathVariable String courseId,
            @PathVariable String videoId) {
        videoService.deleteVideoFromCourse(courseId, videoId);
        return ResponseEntity.noContent().build();
    }
}
```

**Hinglish Comments:**

```java
@RestController
@RequestMapping("/api/v1/courses/{courseId}/videos")
public class CourseVideoController {
    
    // VideoService ko inject karna
    private final VideoService videoService;
    
    // Constructor injection ka use karna
    public CourseVideoController(VideoService videoService) {
        this.videoService = videoService;
    }
    
    // Ek course ke saare videos get karne ke liye
    @GetMapping
    public ResponseEntity<List<VideoDto>> getAllVideosForCourse(@PathVariable String courseId) {
        // courseId ko path se extract karke video service ko pass karna
        return ResponseEntity.ok(videoService.getVideosByCourseId(courseId));
    }
    
    // Ek course ka specific video get karne ke liye
    @GetMapping("/{videoId}")
    public ResponseEntity<VideoDto> getVideoInCourse(
            @PathVariable String courseId,  // Course ID path se
            @PathVariable String videoId) { // Video ID path se
        // Dono IDs ko service ko pass karna
        return ResponseEntity.ok(videoService.getVideoInCourse(courseId, videoId));
    }
    
    // Ek course ke liye naya video create karne ke liye
    @PostMapping
    public ResponseEntity<VideoDto> createVideoForCourse(
            @PathVariable String courseId,  // Course ID path se
            @RequestBody VideoDto videoDto) { // Video data request body se
        // VideoDto mein courseId set karna relationship establish karne ke liye
        videoDto.setCourseId(courseId);
        // 201 Created status ke saath response karna
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(videoService.createVideo(videoDto));
    }
    
    // Course ke existing video ko update karne ke liye
    @PutMapping("/{videoId}")
    public ResponseEntity<VideoDto> updateVideoInCourse(
            @PathVariable String courseId,
            @PathVariable String videoId,
            @RequestBody VideoDto videoDto) {
        // videoDto mein IDs set karna to ensure right video is updated
        videoDto.setVideoId(videoId);
        videoDto.setCourseId(courseId);
        return ResponseEntity.ok(videoService.updateVideo(videoDto));
    }
    
    // Course se video delete karne ke liye
    @DeleteMapping("/{videoId}")
    public ResponseEntity<Void> deleteVideoFromCourse(
            @PathVariable String courseId,
            @PathVariable String videoId) {
        // Service call to delete video, ensuring it belongs to the course
        videoService.deleteVideoFromCourse(courseId, videoId);
        // 204 No Content status return karna
        return ResponseEntity.noContent().build();
    }
}
```

### Service-Level Implementation

Service layer mein nested resources ke liye methods implement karein:

```java
@Service
public class VideoServiceImpl implements VideoService {

    private final VideoRepository videoRepository;
    private final CourseRepository courseRepository;
    private final ModelMapper modelMapper;

    public VideoServiceImpl(
            VideoRepository videoRepository,
            CourseRepository courseRepository,
            ModelMapper modelMapper) {
        this.videoRepository = videoRepository;
        this.courseRepository = courseRepository;
        this.modelMapper = modelMapper;
    }

    @Override
    public List<VideoDto> getVideosByCourseId(String courseId) {
        // Verify course exists
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Get videos for this course
        List<Video> videos = videoRepository.findByCourse(course);
        
        // Convert entities to DTOs
        return videos.stream()
                .map(video -> modelMapper.map(video, VideoDto.class))
                .collect(Collectors.toList());
    }

    @Override
    public VideoDto getVideoInCourse(String courseId, String videoId) {
        // Verify course exists
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Find video and verify it belongs to the course
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoId));
        
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video with id " + videoId + " does not belong to course with id " + courseId);
        }
        
        return modelMapper.map(video, VideoDto.class);
    }

    @Override
    public VideoDto createVideo(VideoDto videoDto) {
        // Validate courseId
        Course course = courseRepository.findById(videoDto.getCourseId())
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + videoDto.getCourseId()));
        
        // Generate a new ID for the video
        String videoId = UUID.randomUUID().toString();
        videoDto.setVideoId(videoId);
        
        // Map DTO to entity
        Video video = modelMapper.map(videoDto, Video.class);
        video.setCourse(course);
        
        // Save entity
        Video savedVideo = videoRepository.save(video);
        
        // Map entity back to DTO
        return modelMapper.map(savedVideo, VideoDto.class);
    }

    @Override
    public VideoDto updateVideo(VideoDto videoDto) {
        // Validate courseId
        Course course = courseRepository.findById(videoDto.getCourseId())
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + videoDto.getCourseId()));
        
        // Validate videoId and check if it belongs to the course
        Video video = videoRepository.findById(videoDto.getVideoId())
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoDto.getVideoId()));
        
        if (!video.getCourse().getId().equals(videoDto.getCourseId())) {
            throw new ResourceNotFoundException("Video with id " + videoDto.getVideoId() + 
                    " does not belong to course with id " + videoDto.getCourseId());
        }
        
        // Update entity fields from DTO
        modelMapper.map(videoDto, video);
        video.setCourse(course);
        
        // Save updated entity
        Video updatedVideo = videoRepository.save(video);
        
        // Map entity back to DTO
        return modelMapper.map(updatedVideo, VideoDto.class);
    }

    @Override
    public void deleteVideoFromCourse(String courseId, String videoId) {
        // Validate courseId
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Validate videoId and check if it belongs to the course
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoId));
        
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video with id " + videoId + 
                    " does not belong to course with id " + courseId);
        }
        
        // Delete the video
        videoRepository.delete(video);
    }
}
```

**Hinglish Comments:**

```java
@Service
public class VideoServiceImpl implements VideoService {

    // Repositories aur ModelMapper inject karna
    private final VideoRepository videoRepository;
    private final CourseRepository courseRepository;
    private final ModelMapper modelMapper;

    // Constructor injection ka use karna
    public VideoServiceImpl(
            VideoRepository videoRepository,
            CourseRepository courseRepository,
            ModelMapper modelMapper) {
        this.videoRepository = videoRepository;
        this.courseRepository = courseRepository;
        this.modelMapper = modelMapper;
    }

    @Override
    public List<VideoDto> getVideosByCourseId(String courseId) {
        // Pehle check karna ki course exist karta hai ya nahi
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Is course ke saare videos fetch karna
        List<Video> videos = videoRepository.findByCourse(course);
        
        // Entity objects ko DTO objects mein convert karna
        return videos.stream()
                .map(video -> modelMapper.map(video, VideoDto.class))
                .collect(Collectors.toList());
    }

    @Override
    public VideoDto getVideoInCourse(String courseId, String videoId) {
        // Course exist karta hai ya nahi, check karna
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Video fetch karna aur verify karna ki woh course se belong karta hai
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoId));
        
        // Check if video belongs to the specified course
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video with id " + videoId + " does not belong to course with id " + courseId);
        }
        
        // Entity ko DTO mein convert karke return karna
        return modelMapper.map(video, VideoDto.class);
    }

    @Override
    public VideoDto createVideo(VideoDto videoDto) {
        // Course ID validate karna
        Course course = courseRepository.findById(videoDto.getCourseId())
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + videoDto.getCourseId()));
        
        // Video ke liye new UUID generate karna
        String videoId = UUID.randomUUID().toString();
        videoDto.setVideoId(videoId);
        
        // DTO ko entity mein convert karna
        Video video = modelMapper.map(videoDto, Video.class);
        video.setCourse(course);  // Course relationship set karna
        
        // Entity ko database mein save karna
        Video savedVideo = videoRepository.save(video);
        
        // Saved entity ko DTO mein convert karke return karna
        return modelMapper.map(savedVideo, VideoDto.class);
    }

    @Override
    public VideoDto updateVideo(VideoDto videoDto) {
        // Course ID validate karna
        Course course = courseRepository.findById(videoDto.getCourseId())
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + videoDto.getCourseId()));
        
        // Video ID validate karna aur check karna ki woh course se belong karta hai
        Video video = videoRepository.findById(videoDto.getVideoId())
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoDto.getVideoId()));
        
        // Verify video belongs to the specified course
        if (!video.getCourse().getId().equals(videoDto.getCourseId())) {
            throw new ResourceNotFoundException("Video with id " + videoDto.getVideoId() + 
                    " does not belong to course with id " + videoDto.getCourseId());
        }
        
        // DTO se entity ke fields update karna
        modelMapper.map(videoDto, video);
        video.setCourse(course);  // Ensure course relationship is maintained
        
        // Updated entity ko save karna
        Video updatedVideo = videoRepository.save(video);
        
        // Entity ko DTO mein convert karke return karna
        return modelMapper.map(updatedVideo, VideoDto.class);
    }

    @Override
    public void deleteVideoFromCourse(String courseId, String videoId) {
        // Course ID validate karna
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Video ID validate karna aur check karna ki woh course se belong karta hai
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found with id: " + videoId));
        
        // Verify video belongs to the specified course
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video with id " + videoId + 
                    " does not belong to course with id " + courseId);
        }
        
        // Video ko delete karna
        videoRepository.delete(video);
    }
}
```

### Repository-Level Implementation

Repository layer mein nested resources ke querying ko support karne ke liye methods add karein:

```java
public interface VideoRepository extends JpaRepository<Video, String> {
    List<Video> findByCourse(Course course);
    List<Video> findByCourseId(String courseId);
    // For more complex queries:
    @Query("SELECT v FROM Video v WHERE v.course.id = :courseId AND v.title LIKE %:keyword%")
    List<Video> searchVideosInCourse(@Param("courseId") String courseId, @Param("keyword") String keyword);
}
```

## 4. Entity Relationships for Nested Resources

### Parent-Child Relationship Setup

JPA annotations ke saath entity relationships define karein:

**Course Entity (Parent):**

```java
@Entity
@Table(name = "courses")
@Data
public class Course {
    @Id
    private String id;
    private String title;
    private String shortDesc;
    @Column(length = 2000)
    private String longDesc;
    private double price;
    private boolean live = false;
    private double discount;
    private Date createdDate;
    
    // Banner fields
    private String banner;
    private String bannerContentType;
    
    // One-to-Many relationship with videos
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Video> videos = new ArrayList<>();
    
    // Helper methods for managing videos
    public void addVideo(Video video) {
        videos.add(video);
        video.setCourse(this);
    }
    
    public void removeVideo(Video video) {
        videos.remove(video);
        video.setCourse(null);
    }
}
```

**Video Entity (Child):**

```java
@Entity
@Table(name = "videos")
@Data
public class Video {
    @Id
    private String videoId;
    private String title;
    @Column(length = 1000)
    private String description;
    private String filePath;
    private String contentType;
    
    // Many-to-One relationship with course
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "course_id")
    private Course course;
}
```

### DTO Classes for Nested Resources

DTOs create karein jo nested resources ko represent karein:

**CourseDto with VideoDto list:**

```java
@Data
public class CourseDto {
    private String id;
    private String title;
    private String shortDesc;
    private String longDesc;
    private double price;
    private boolean live;
    private double discount;
    private Date createdDate;
    private String banner;
    private String bannerContentType;
    
    // Videos collection for nested resource representation
    private List<VideoDto> videos;
}
```

**VideoDto with courseId reference:**

```java
@Data
public class VideoDto {
    private String videoId;
    private String title;
    private String description;
    private String filePath;
    private String contentType;
    
    // Reference to parent by ID
    private String courseId;
}
```

## 5. Project Example: E-Learning Platform

Our E-Learning platform mein humne already dekhte hain ki Course aur Video ke beech parent-child relationship hai. CourseController mein `/api/v1/courses/{courseId}/banners` endpoint hai, jo nested resource ka example hai.

Ab hum `/api/v1/courses/{courseId}/videos` endpoints implement karenge.

### Step 1: CourseVideoController Create Karna

```java
@RestController
@RequestMapping("/api/v1/courses/{courseId}/videos")
public class CourseVideoController {
    
    private final VideoService videoService;
    
    @Autowired
    public CourseVideoController(VideoService videoService) {
        this.videoService = videoService;
    }
    
    @GetMapping
    public ResponseEntity<List<VideoDto>> getVideosForCourse(@PathVariable String courseId) {
        return ResponseEntity.ok(videoService.getVideosByCourseId(courseId));
    }
    
    @PostMapping
    public ResponseEntity<VideoDto> addVideoToCourse(
            @PathVariable String courseId,
            @RequestBody VideoDto videoDto) {
        videoDto.setCourseId(courseId);
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(videoService.createVideo(videoDto));
    }
    
    @GetMapping("/{videoId}")
    public ResponseEntity<VideoDto> getVideoFromCourse(
            @PathVariable String courseId,
            @PathVariable String videoId) {
        return ResponseEntity.ok(videoService.getVideoInCourse(courseId, videoId));
    }
    
    @PutMapping("/{videoId}")
    public ResponseEntity<VideoDto> updateVideoInCourse(
            @PathVariable String courseId,
            @PathVariable String videoId,
            @RequestBody VideoDto videoDto) {
        videoDto.setVideoId(videoId);
        videoDto.setCourseId(courseId);
        return ResponseEntity.ok(videoService.updateVideo(videoDto));
    }
    
    @DeleteMapping("/{videoId}")
    public ResponseEntity<Void> removeVideoFromCourse(
            @PathVariable String courseId,
            @PathVariable String videoId) {
        videoService.deleteVideoFromCourse(courseId, videoId);
        return ResponseEntity.noContent().build();
    }
}
```

### Step 2: VideoService Interface Extend Karna

```java
public interface VideoService {
    // Existing methods
    VideoDto createVideo(VideoDto videoDto);
    VideoDto updateVideo(VideoDto videoDto);
    VideoDto getVideoById(String id);
    Page<VideoDto> getAllVideos(Pageable pageable);
    void deleteVideo(String id);
    List<VideoDto> searchVideos(String keyword);
    
    // New methods for nested resources
    List<VideoDto> getVideosByCourseId(String courseId);
    VideoDto getVideoInCourse(String courseId, String videoId);
    void deleteVideoFromCourse(String courseId, String videoId);
}
```

### Step 3: VideoServiceImpl Update Karna

```java
@Service
public class VideoServiceImpl implements VideoService {
    
    // Existing implementations...
    
    @Override
    public List<VideoDto> getVideosByCourseId(String courseId) {
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        List<Video> videos = videoRepository.findByCourse(course);
        
        return videos.stream()
                .map(video -> {
                    VideoDto dto = modelMapper.map(video, VideoDto.class);
                    dto.setCourseId(courseId);
                    return dto;
                })
                .collect(Collectors.toList());
    }
    
    @Override
    public VideoDto getVideoInCourse(String courseId, String videoId) {
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found"));
        
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found"));
        
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video does not belong to specified course");
        }
        
        VideoDto dto = modelMapper.map(video, VideoDto.class);
        dto.setCourseId(courseId);
        return dto;
    }
    
    @Override
    public void deleteVideoFromCourse(String courseId, String videoId) {
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found"));
        
        Video video = videoRepository.findById(videoId)
                .orElseThrow(() -> new ResourceNotFoundException("Video not found"));
        
        if (!video.getCourse().getId().equals(courseId)) {
            throw new ResourceNotFoundException("Video does not belong to specified course");
        }
        
        videoRepository.delete(video);
    }
}
```

## 6. Testing Nested Resource Endpoints with Postman/cURL

### Course Videos List API Test

#### Request:
```http
GET /api/v1/courses/c001/videos
Accept: application/json
```

#### cURL:
```bash
curl -X GET "http://localhost:8081/api/v1/courses/c001/videos" -H "accept: application/json"
```

#### Expected Response:
```json
[
  {
    "videoId": "v001",
    "title": "Introduction to Spring Boot",
    "description": "Learn the basics of Spring Boot",
    "courseId": "c001"
  },
  {
    "videoId": "v002",
    "title": "RESTful APIs with Spring Boot",
    "description": "Creating REST APIs with Spring Boot",
    "courseId": "c001"
  }
]
```

### Add Video to Course API Test

#### Request:
```http
POST /api/v1/courses/c001/videos
Content-Type: application/json

{
  "title": "Working with Nested Resources",
  "description": "Learn how to implement nested resources in Spring Boot"
}
```

#### cURL:
```bash
curl -X POST "http://localhost:8081/api/v1/courses/c001/videos" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Working with Nested Resources",
    "description": "Learn how to implement nested resources in Spring Boot"
  }'
```

#### Expected Response:
```json
{
  "videoId": "v003",
  "title": "Working with Nested Resources",
  "description": "Learn how to implement nested resources in Spring Boot",
  "courseId": "c001"
}
```

### Get Video in Course API Test

#### Request:
```http
GET /api/v1/courses/c001/videos/v001
Accept: application/json
```

#### cURL:
```bash
curl -X GET "http://localhost:8081/api/v1/courses/c001/videos/v001" -H "accept: application/json"
```

#### Expected Response:
```json
{
  "videoId": "v001",
  "title": "Introduction to Spring Boot",
  "description": "Learn the basics of Spring Boot",
  "courseId": "c001"
}
```

## 7. Best Practices for Nested Resources

### 1. Nesting Levels Ko Limit Karein

Recommended hai ki aap nesting ko 1-2 levels tak hi rakhein, jaise:

```
/courses/{courseId}/videos
```

Deep nesting se endpoints complex ho jaate hain:

```
/courses/{courseId}/videos/{videoId}/comments/{commentId}/replies
```

### 2. Resource IDs Ko Consistent Naming Convention Follow Karein

Path variables ke liye consistent naming convention rakhen:

```
/{resourceName}/{resourceId}/{nestedResourceName}/{nestedResourceId}
```

### 3. Relationship Validation Hamesha Karein

Parent-child relationship valid hai ya nahi, ye validation service layer mein zaroor karein:

```java
if (!video.getCourse().getId().equals(courseId)) {
    throw new ResourceNotFoundException("Video does not belong to the specified course");
}
```

### 4. Alternative Collection Endpoints Provide Karein

Nested resources ke alternative direct endpoints bhi provide karein:

```
GET /videos?courseId={courseId}  // Alternative to /courses/{courseId}/videos
```

### 5. Pagination and Filtering Support Karein

Nested collections mein pagination and filtering support karein:

```
GET /courses/{courseId}/videos?page=0&size=10&sort=title,asc
```

### 6. HATEOAS (Hypertext as the Engine of Application State) Use Karein

HATEOAS principles follow karke related resources ke links provide karein:

```json
{
  "videoId": "v001",
  "title": "Introduction to Spring Boot",
  "description": "Learn the basics of Spring Boot",
  "courseId": "c001",
  "_links": {
    "self": {
      "href": "/api/v1/courses/c001/videos/v001"
    },
    "course": {
      "href": "/api/v1/courses/c001"
    },
    "comments": {
      "href": "/api/v1/courses/c001/videos/v001/comments"
    }
  }
}
```

## 8. Common Issues and Solutions

### 1. Circular Dependencies in JSON Serialization

**Issue**: Entity relationships mein circular references (Course -> Videos -> Course) cause serialization problems.

**Solution**: 
- DTOs use karein jahan circular references na ho
- @JsonIgnore annotation use karein
- Custom serializers implement karein

```java
public class VideoDto {
    private String videoId;
    private String title;
    private String courseId;  // Only ID, not full course object
}
```

### 2. N+1 Query Problem

**Issue**: For each parent, child ko fetch karne ke liye separate query execute hoti hai.

**Solution**:
- JPA fetch joins use karein
- Entity graph annotations use karein

```java
@EntityGraph(attributePaths = {"videos"})
Optional<Course> findById(String id);
```

### 3. URL Length Limitations

**Issue**: Deep nested URLs can become too long.

**Solution**:
- Nesting levels limit karein
- Query parameters use karein

### 4. Resource Ownership and Security

**Issue**: Ensuring that only authorized users can access nested resources.

**Solution**:
- Security checks implement karein jo parent-child relationship verify karein
- Method-level security with SpEL expressions:

```java
@PreAuthorize("@securityService.canAccessCourse(#courseId)")
public List<VideoDto> getVideosByCourseId(String courseId) {
    // ...
}
```

## 9. Interviewer Q/A on Nested Resources

### Q1: REST mein nested resources ka kya use hai?

**A:** Nested resources ka use real-world object relationships ko represent karne ke liye hota hai. For example, ek Course ke andar Videos, ya ek User ke posts. Ye RESTful API design ko more intuitive banata hai and allows developers to clearly express hierarchical relationships. Nested resources se client ko related resources access karne ke liye endpoints zyada meaningful mil jaate hain.

### Q2: Kya aaap /courses/{courseId}/videos ki jagah /videos?courseId={courseId} use kar sakte hain?

**A:** Haan, dono approaches valid hain, but different scenarios mein useful hain:

- `/courses/{courseId}/videos` is good when the relationship is a strong parent-child relationship and videos are viewed as sub-resources of a course.
- `/videos?courseId={courseId}` is better when you need to filter videos by different criteria and course is just one of many possible filters.

Best practice is to offer both: nested resource endpoints for hierarchical access and query parameter-based filtering for more flexible searches.

### Q3: Nested resources ka use karte waqt kaise aap circular references se bachte hain?

**A:** Circular references se bachne ke liye:

1. DTOs use karein jo only necessary fields aur references contain karte hain
2. Entity class mein @JsonIgnore annotation use karein to prevent serializing a back-reference
3. Implement custom serializers jo circular references handle karte hain
4. Reference by ID instead of embedding complete objects

Example mein, VideoDto mein course ka pura object ki jagah sirf courseId rakhte hain.

### Q4: Kaise aap parent resource ke delete hone par child resources handle karte hain?

**A:** Child resources ko handle karne ke liye multiple strategies hain:

1. **Cascade Delete**: JPA's `CascadeType.REMOVE` or `orphanRemoval=true` use karke automatically delete child resources
   ```java
   @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
   private List<Video> videos;
   ```

2. **Set Null/Default Reference**: Foreign key constraint ko optional bana dein and set NULL on parent delete
   ```java
   @ManyToOne(optional = true)
   @JoinColumn(name = "course_id")
   private Course course;
   ```

3. **Prevent Deletion**: Check for existing children and prevent the deletion of a parent if children exist
   ```java
   if (!course.getVideos().isEmpty()) {
       throw new IllegalOperationException("Cannot delete course with existing videos");
   }
   ```

4. **Transfer Ownership**: Move children to another parent

Strategy selection depends on your business requirements.

### Q5: Spring Boot mein nested resources ke authentication aur authorization kaise handle karte hain?

**A:** Authentication and authorization ke liye:

1. **Method-Level Security**:
   ```java
   @PreAuthorize("@securityService.canAccessCourse(#courseId)")
   public List<VideoDto> getVideosByCourseId(String courseId) {
       // ...
   }
   ```

2. **Custom SecurityExpressions**:
   ```java
   public class SecurityService {
       public boolean canAccessCourse(String courseId) {
           // Check if current user has access to the course
       }
   }
   ```

3. **Resource Ownership Validation**: Service layer mein check karein ki resource current user se belong karta hai ya nahi

4. **Hierarchical Access Control**: Ensure that user who has access to a child resource also has access to the parent

### Q6: Kya aap multiple levels of nesting recommend karte hain? For example, /courses/{courseId}/videos/{videoId}/comments/{commentId}

**A:** General best practice is to limit nesting to 1-2 levels maximum. Deep nesting can lead to:

- Very long URLs
- Complexity in maintaining and documenting APIs
- Difficulty in understanding the API structure

For deeper relationships, alternatives hain:
- Use query parameters: `/comments?videoId={videoId}`
- Create separate endpoints: `/video-comments/{videoId}`
- Implement HATEOAS to provide navigation links

If you do need deeper nesting, make sure to also provide flatter alternatives for flexibility.

### Q7: Nested resources ke context mein HTTP status codes ka appropriate use kya hai?

**A:** Key status codes for nested resources:

- **200 OK**: Successful GET of resources
- **201 Created**: Successfully created a nested resource
- **204 No Content**: Successfully deleted a nested resource
- **400 Bad Request**: Invalid input for creating/updating a nested resource
- **404 Not Found**: When either parent or child resource doesn't exist
- **409 Conflict**: When trying to create a duplicate nested resource
- **422 Unprocessable Entity**: When the provided data is valid but can't be processed (e.g., adding a video to a course that is already full)

Especially important for nested resources is **404 Not Found** when the parent resource doesn't exist or when the child resource doesn't exist within that specific parent.

## 10. Real-world Use Cases

### 1. E-Learning Platform (Our Example)
- `/courses/{courseId}/videos`: Videos belonging to a course
- `/courses/{courseId}/students`: Students enrolled in a course
- `/courses/{courseId}/assignments`: Assignments for a course

### 2. E-Commerce Platform
- `/products/{productId}/reviews`: Reviews for a product
- `/orders/{orderId}/items`: Items in an order
- `/users/{userId}/addresses`: Addresses of a user

### 3. Social Media Platform
- `/users/{userId}/posts`: Posts by a user
- `/posts/{postId}/comments`: Comments on a post
- `/users/{userId}/friends`: Friends of a user

### 4. Content Management System
- `/articles/{articleId}/sections`: Sections within an article
- `/categories/{categoryId}/articles`: Articles in a category
- `/articles/{articleId}/tags`: Tags for an article

## 11. Text Diagram: Nested Resources Architecture

```
┌─────────────────────────┐         ┌─────────────────────────┐
│                         │         │                         │
│      REST Controller    │ ◄─────► │      Service Layer      │
│                         │         │                         │
└─────────────┬───────────┘         └─────────────┬───────────┘
              │                                   │
              │                                   │
              ▼                                   ▼
┌─────────────────────────┐         ┌─────────────────────────┐
│                         │         │                         │
│     URL Structure       │         │    Entity Relations     │
│                         │         │                         │
└─────────────────────────┘         └─────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ URL Structure Examples:                                      │
│                                                             │
│ /courses                        (All courses)               │
│ /courses/{courseId}             (Single course)             │
│ /courses/{courseId}/videos      (Videos of a course)        │
│ /courses/{courseId}/videos/{id} (Single video of a course)  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Entity Relations:                                           │
│                                                             │
│ Course ──┐                                                  │
│          │ @OneToMany                                       │
│          ▼                                                  │
│ Video ───┘ @ManyToOne                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Request Flow:                                               │
│                                                             │
│ Client ──► Controller ──► Service ──► Repository ──► DB     │
│          (Extracts     (Validates    (Queries      (Stores  │
│           path vars)    relationship)  data)        data)   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Conclusion

Nested resources REST APIs mein real-world relationships ko express karne ka ek powerful tarika hain. Spring Boot mein nested resources implement karne ke liye proper controller structure, service layer validation, aur entity relationships ki zarurat hai.

Best practices ko follow karke - jaise ki nesting levels ko limit karna, proper validation implement karna, aur alternative access patterns provide karna - aap ek well-designed, intuitive, aur maintainable API create kar sakte hain jo complex data relationships ko handle karta hai.

Nested resources implement karte waqt remember karein ki design simplicity, usability, aur performance ke beech balance maintain karna zaroori hai.
