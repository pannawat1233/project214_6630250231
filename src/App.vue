<template>
  <div class="app-container" :class="{ 'dark': true }">
    <header class="header">
      <div class="gradient-animation"></div>
      <div class="container">
        <div class="header-content">
          <h1 class="logo">
            <span class="logo-text">STUDENT</span>
            <span class="logo-accent">PORTAL</span>
          </h1>
          <div class="header-actions">
            <button class="theme-button">
              <span class="theme-icon">🌙</span>
            </button>
          </div>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div class="container">
        <section class="profile-section">
          <div class="glass-card profile-card">
            <div class="card-glow"></div>
            <div class="profile-header">
              <div class="profile-image-container">
                <img src="/img/88254282237-_1oo.jpg" alt="Student Photo" class="profile-image" />
                <div class="profile-image-overlay">
                  <button class="edit-photo-btn" @click="showPhotoEditModal = true">
                    <i class="icon">📷</i>
                  </button>
                </div>
              </div>
              <div class="profile-info">
                <h2 class="student-name">{{ student.name }}</h2>
                <p class="student-id">รหัสนิสิต: {{ student.studentId }}</p>
                <p class="student-major">{{ student.major }}</p>
                <div class="status-badge">{{ student.status }}</div>
              </div>
              <button class="neon-button edit-profile-btn" @click="showProfileEditModal = true">
                <span class="button-text">แก้ไขข้อมูล</span>
                <span class="button-icon">✏️</span>
              </button>
            </div>
            
            <div class="profile-details">
              <div class="detail-item">
                <span class="detail-label">โรงเรียนเดิม</span>
                <span class="detail-value">{{ student.previousSchool }}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">คณะ</span>
                <span class="detail-value">{{ student.faculty }}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">ภาควิชา</span>
                <span class="detail-value">{{ student.department }}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">ปีการศึกษาที่เข้า</span>
                <span class="detail-value">{{ student.enrollmentYear }}</span>
              </div>
            </div>
          </div>
        </section>

        <section class="academic-section">
          <div class="section-header">
            <h2 class="section-title">รายละเอียดการเรียน</h2>
            <button class="neon-button add-course-btn" @click="showAddCourseModal = true">
              <span class="button-text">เพิ่มรายวิชา</span>
              <span class="button-icon">➕</span>
            </button>
          </div>

          <div class="academic-summary">
            <div class="glass-card summary-item">
              <div class="summary-icon">📊</div>
              <span class="summary-value">{{ calculateGPA() }}</span>
              <span class="summary-label">เกรดเฉลี่ย</span>
            </div>
            <div class="glass-card summary-item">
              <div class="summary-icon">🎓</div>
              <span class="summary-value">{{ calculateTotalCredits() }}</span>
              <span class="summary-label">หน่วยกิตทั้งหมด</span>
            </div>
            <div class="glass-card summary-item">
              <div class="summary-icon">📚</div>
              <span class="summary-value">{{ courses.length }}</span>
              <span class="summary-label">รายวิชา</span>
            </div>
          </div>

          <div class="glass-card courses-table-container">
            <div class="card-glow"></div>
            <table class="courses-table">
              <thead>
                <tr>
                  <th>รหัสวิชา</th>
                  <th>ชื่อวิชา</th>
                  <th>หน่วยกิต</th>
                  <th>เกรด</th>
                  <th>การจัดการ</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="course in courses" :key="course.id" class="course-row">
                  <td>{{ course.code }}</td>
                  <td>{{ course.name }}</td>
                  <td>{{ course.credits }}</td>
                  <td>
                    <span class="grade" :class="getGradeClass(course.grade)">
                      {{ course.grade }}
                    </span>
                  </td>
                  <td class="actions">
                    <button class="action-btn edit-btn" @click="editCourse(course)">
                      <i class="icon">✏️</i>
                    </button>
                    <button class="action-btn delete-btn" @click="deleteCourse(course.id)">
                      <i class="icon">🗑️</i>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

        <section class="carousel-section">
          <div class="carousel-container">
            <div class="carousel-slide" v-for="(image, index) in carouselImages" :key="index" :class="{ 'active': currentSlide === index }">
              <img :src="image.src" :alt="image.alt" class="carousel-image" />
              <div class="carousel-caption">
                <h3>{{ image.title }}</h3>
                <p>{{ image.description }}</p>
              </div>
            </div>
            <button class="carousel-control prev" @click="prevSlide">❮</button>
            <button class="carousel-control next" @click="nextSlide">❯</button>
            <div class="carousel-indicators">
              <button 
                v-for="(image, index) in carouselImages" 
                :key="index" 
                class="indicator" 
                :class="{ 'active': currentSlide === index }"
                @click="goToSlide(index)">
              </button>
            </div>
          </div>
        </section>
      </div>
    </main>

    <div class="modal" v-if="showProfileEditModal">
      <div class="modal-content glass-modal">
        <div class="modal-header">
          <h3>แก้ไขข้อมูลนิสิต</h3>
          <button class="close-btn" @click="showProfileEditModal = false">×</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="updateProfile">
            <div class="form-group">
              <label for="name">ชื่อ-นามสกุล</label>
              <input type="text" id="name" v-model="editedStudent.name" required />
            </div>
            <div class="form-group">
              <label for="studentId">รหัสนิสิต</label>
              <input type="text" id="studentId" v-model="editedStudent.studentId" required />
            </div>
            <div class="form-group">
              <label for="major">สาขา</label>
              <input type="text" id="major" v-model="editedStudent.major" required />
            </div>
            <div class="form-group">
              <label for="previousSchool">โรงเรียนเดิม</label>
              <input type="text" id="previousSchool" v-model="editedStudent.previousSchool" required />
            </div>
            <div class="form-group">
              <label for="faculty">คณะ</label>
              <input type="text" id="faculty" v-model="editedStudent.faculty" required />
            </div>
            <div class="form-group">
              <label for="department">ภาควิชา</label>
              <input type="text" id="department" v-model="editedStudent.department" required />
            </div>
            <div class="form-group">
              <label for="enrollmentYear">ปีการศึกษาที่เข้า</label>
              <input type="text" id="enrollmentYear" v-model="editedStudent.enrollmentYear" required />
            </div>
            <div class="form-group">
              <label for="status">สถานภาพ</label>
              <input type="text" id="status" v-model="editedStudent.status" required />
            </div>
            <div class="form-actions">
              <button type="button" class="cancel-btn" @click="showProfileEditModal = false">ยกเลิก</button>
              <button type="submit" class="neon-button save-btn">บันทึก</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <div class="modal" v-if="showPhotoEditModal">
      <div class="modal-content glass-modal">
        <div class="modal-header">
          <h3>เปลี่ยนรูปภาพ</h3>
          <button class="close-btn" @click="showPhotoEditModal = false">×</button>
        </div>
        <div class="modal-body">
          <div class="photo-upload-container">
            <div class="current-photo">
              <img :src="student.imageUrl || '/img/88254282237-_1oo.jpg'" alt="Current Photo" />
            </div>
            <div class="upload-controls">
              <input type="file" id="photo-upload" @change="handlePhotoUpload" accept="image/*" />
              <label for="photo-upload" class="neon-button upload-btn">เลือกรูปภาพ</label>
              <p class="upload-hint">รูปภาพตัวเองเท่านั้น (เห็นหน้าจะได้รู้ว่าใคร)</p>
            </div>
          </div>
          <div class="form-actions">
            <button type="button" class="cancel-btn" @click="showPhotoEditModal = false">ยกเลิก</button>
            <button type="button" class="neon-button save-btn" @click="updatePhoto">บันทึก</button>
          </div>
        </div>
      </div>
    </div>

    <div class="modal" v-if="showAddCourseModal || showEditCourseModal">
      <div class="modal-content glass-modal">
        <div class="modal-header">
          <h3>{{ showAddCourseModal ? 'เพิ่มรายวิชา' : 'แก้ไขรายวิชา' }}</h3>
          <button class="close-btn" @click="closeCourseForms">×</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="showAddCourseModal ? addCourse() : updateCourse()">
            <div class="form-group">
              <label for="courseCode">รหัสวิชา</label>
              <input type="text" id="courseCode" v-model="editedCourse.code" required />
            </div>
            <div class="form-group">
              <label for="courseName">ชื่อวิชา</label>
              <input type="text" id="courseName" v-model="editedCourse.name" required />
            </div>
            <div class="form-group">
              <label for="courseCredits">หน่วยกิต</label>
              <input type="number" id="courseCredits" v-model="editedCourse.credits" min="1" max="6" required />
            </div>
            <div class="form-group">
              <label for="courseGrade">เกรด</label>
              <select id="courseGrade" v-model="editedCourse.grade" required>
                <option value="A">A</option>
                <option value="B+">B+</option>
                <option value="B">B</option>
                <option value="C+">C+</option>
                <option value="C">C</option>
                <option value="D+">D+</option>
                <option value="D">D</option>
                <option value="F">F</option>
                <option value="W">W</option>
                <option value="I">I</option>
              </select>
            </div>
            <div class="form-actions">
              <button type="button" class="cancel-btn" @click="closeCourseForms">ยกเลิก</button>
              <button type="submit" class="neon-button save-btn">บันทึก</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <div class="modal" v-if="showConfirmModal">
      <div class="modal-content glass-modal confirmation-modal">
        <div class="modal-header">
          <h3>ยืนยันการลบ</h3>
          <button class="close-btn" @click="showConfirmModal = false">×</button>
        </div>
        <div class="modal-body">
          <div class="confirm-icon">🗑️</div>
          <p>คุณต้องการลบรายวิชานี้ใช่หรือไม่?</p>
          <div class="form-actions">
            <button type="button" class="cancel-btn" @click="showConfirmModal = false">ยกเลิก</button>
            <button type="button" class="delete-confirm-btn" @click="confirmDelete">ลบ</button>
          </div>
        </div>
      </div>
    </div>

    <div class="sidebar" :class="{ 'active': showSidebar }">
      <div class="sidebar-header">
        <div class="profile">
          <img class="profile-img" src="/img/88254282237-_1oo.jpg" alt="profile" />
          <div class="profile-info">
            <p>{{ student.name }}</p>
            <p class="subtext">{{ student.status }}</p>
          </div>
        </div>
        <button class="close-sidebar" @click="toggleSidebar">×</button>
      </div>
      <ul class="menu">
        <li class="menu-item active">
          <i class="icon">📊</i> ข้อมูลนิสิต
        </li>
      </ul>
    </div>

    <button class="menu-toggle" @click="toggleSidebar">
      <span class="menu-icon">☰</span>
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isDarkMode: true,
      showSidebar: false,
      currentSlide: 0,
      carouselImages: [
        {
          src: "/img/517f4336e4a3819dee6f9ec4d18fb94c_3292181308204717228.png",
          alt: "Campus Image 1",
          title: "Kasetsart University",
          description: "Welcome to our beautiful campus"
        },
        {
          src: "/img/4f33f8d9643641e5eaf9a32098a3052e_6327073662518545639.png",
          alt: "Campus Image 2",
          title: "Student Life",
          description: "Explore the vibrant student community"
        },
        {
          src: "/img/3aacb552be7ed7960153458cbcda1a78_6042781602698052612.png",
          alt: "Campus Image 3",
          title: "Academic Excellence",
          description: "Pursue your academic goals with us"
        }
      ],
      student: {
        id: 1,
        name: "ปัณณวัฒน์ นิ่งเจริญ",
        studentId: "6630250231",
        major: "วิทยาการคอมพิวเตอร์ (ภาคพิเศษ)",
        previousSchool: "มหาวิทยาลัยเกษตรศาสตร์",
        faculty: "วิทยาศาสตร์ ศรีราชา",
        department: "วิทยาการคอมพิวเตอร์",
        enrollmentYear: "2568",
        status: "นิสิตปัจจุบัน",
        imageUrl: "/img/88254282237-_1oo.jpg"
      },
      editedStudent: {},
      courses: [
        { id: 1, code: "01418211", name: "การโปรแกรมภาษาเชิงวัตถุ", credits: 3, grade: "A" },
        { id: 2, code: "01418231", name: "โครงสร้างข้อมูล", credits: 3, grade: "B+" },
        { id: 3, code: "01418232", name: "การออกแบบและวิเคราะห์ขั้นตอนวิธี", credits: 3, grade: "B" },
        { id: 4, code: "01418321", name: "ระบบฐานข้อมูล", credits: 3, grade: "A" }
      ],
      editedCourse: {
        id: null,
        code: "",
        name: "",
        credits: 3,
        grade: "A"
      },
      showProfileEditModal: false,
      showPhotoEditModal: false,
      showAddCourseModal: false,
      showEditCourseModal: false,
      showConfirmModal: false,
      courseToDeleteId: null,
      uploadedPhoto: null
    };
  },
  created() {
    this.fetchStudentData();
    this.fetchCourses();
    
    this.startCarousel();
  },
  beforeUnmount() {
    this.stopCarousel();
  },
  methods: {
    toggleTheme() {
      this.isDarkMode = !this.isDarkMode;
      document.body.classList.toggle('dark', this.isDarkMode);
    },
    
    toggleSidebar() {
      this.showSidebar = !this.showSidebar;
    },
    
    startCarousel() {
      this.carouselInterval = setInterval(() => {
        this.nextSlide();
      }, 5000);
    },
    
    stopCarousel() {
      clearInterval(this.carouselInterval);
    },
    
    nextSlide() {
      this.currentSlide = (this.currentSlide + 1) % this.carouselImages.length;
    },
    
    prevSlide() {
      this.currentSlide = (this.currentSlide - 1 + this.carouselImages.length) % this.carouselImages.length;
    },
    
    goToSlide(index) {
      this.currentSlide = index;
    },
    
    async fetchStudentData() {
      try {
        const response = await fetch('http://localhost:3000/student');
        const data = await response.json();
        if (data) {
          this.student = data;
        }
      } catch (error) {
        console.error('Error fetching student data:', error);
      }
    },
    
    async fetchCourses() {
      try {
        const response = await fetch('http://localhost:3000/courses');
        const data = await response.json();
        if (data) {
          this.courses = data;
        }
      } catch (error) {
        console.error('Error fetching courses:', error);
      }
    },
    
    async updateProfile() {
      try {
        await fetch(`http://localhost:3000/student`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(this.editedStudent),
        });
        
        this.student = { ...this.editedStudent };
        this.showProfileEditModal = false;
      } catch (error) {
        console.error('Error updating profile:', error);
      }
    },
    
    async updatePhoto() {
      if (!this.uploadedPhoto) return;
      
      try {
        const updatedStudent = { ...this.student, imageUrl: this.uploadedPhoto };
        
        await fetch(`http://localhost:3000/student`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(updatedStudent),
        });
        
        this.student.imageUrl = this.uploadedPhoto;
        this.showPhotoEditModal = false;
      } catch (error) {
        console.error('Error updating photo:', error);
      }
    },
    
    async addCourse() {
      try {
        const newCourse = {
          ...this.editedCourse,
          id: Date.now()
        };
        
        const response = await fetch('http://localhost:3000/courses', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(newCourse),
        });
        
        const savedCourse = await response.json();
        this.courses.push(savedCourse);
        this.resetCourseForm();
        this.showAddCourseModal = false;
      } catch (error) {
        console.error('Error adding course:', error);
      }
    },
    
    async updateCourse() {
      try {
        await fetch(`http://localhost:3000/courses/${this.editedCourse.id}`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(this.editedCourse),
        });
        
        const index = this.courses.findIndex(c => c.id === this.editedCourse.id);
        if (index !== -1) {
          this.courses[index] = { ...this.editedCourse };
        }
        
        this.resetCourseForm();
        this.showEditCourseModal = false;
      } catch (error) {
        console.error('Error updating course:', error);
      }
    },
    
    async deleteCourseFromServer(id) {
      try {
        await fetch(`http://localhost:3000/courses/${id}`, {
          method: 'DELETE',
        });
        
        this.courses = this.courses.filter(c => c.id !== id);
      } catch (error) {
        console.error('Error deleting course:', error);
      }
    },
    
    editProfile() {
      this.editedStudent = { ...this.student };
      this.showProfileEditModal = true;
    },
    
    handlePhotoUpload(event) {
      const file = event.target.files[0];
      if (file) {
        
        const reader = new FileReader();
        reader.onload = (e) => {
          this.uploadedPhoto = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    
    editCourse(course) {
      this.editedCourse = { ...course };
      this.showEditCourseModal = true;
    },
    
    deleteCourse(id) {
      this.courseToDeleteId = id;
      this.showConfirmModal = true;
    },
    
    confirmDelete() {
      if (this.courseToDeleteId) {
        this.deleteCourseFromServer(this.courseToDeleteId);
        this.courseToDeleteId = null;
        this.showConfirmModal = false;
      }
    },
    
    closeCourseForms() {
      this.showAddCourseModal = false;
      this.showEditCourseModal = false;
      this.resetCourseForm();
    },
    
    resetCourseForm() {
      this.editedCourse = {
        id: null,
        code: "",
        name: "",
        credits: 3,
        grade: "A"
      };
    },
    
    calculateGPA() {
      if (this.courses.length === 0) return "0.00";
      
      const gradePoints = {
        'A': 4.0,
        'B+': 3.5,
        'B': 3.0,
        'C+': 2.5,
        'C': 2.0,
        'D+': 1.5,
        'D': 1.0,
        'F': 0.0
      };
      
      let totalPoints = 0;
      let totalCredits = 0;
      
      this.courses.forEach(course => {
        if (course.grade in gradePoints) {
          totalPoints += gradePoints[course.grade] * course.credits;
          totalCredits += course.credits;
        }
      });
      
      return totalCredits > 0 ? (totalPoints / totalCredits).toFixed(2) : "0.00";
    },
    
    calculateTotalCredits() {
      return this.courses.reduce((sum, course) => sum + course.credits, 0);
    },
    
    getGradeClass(grade) {
      if (grade === 'A' || grade === 'B+' || grade === 'B') return 'grade-good';
      if (grade === 'C+' || grade === 'C') return 'grade-average';
      if (grade === 'D+' || grade === 'D') return 'grade-warning';
      if (grade === 'F') return 'grade-fail';
      return 'grade-other';
    }
  }
};
</script>

<style>
:root {
  --primary-color: #00ff95;
  --primary-hover: #00cc76;
  --primary-glow: rgba(0, 255, 149, 0.5);
  --secondary-color: #ff00c8;
  --secondary-glow: rgba(255, 0, 200, 0.5);
  --success-color: #00ff95;
  --warning-color: #ffcc00;
  --danger-color: #ff3366;
  --text-color: #ffffff;
  --text-secondary: #a0a0a0;
  --bg-color: #0a0a0a;
  --card-bg: #151515;
  --card-bg-hover: #1a1a1a;
  --border-color: #333333;
  --glass-bg: rgba(25, 25, 25, 0.8);
  --glass-border: rgba(255, 255, 255, 0.1);
  --shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
  --glow-shadow: 0 0 15px var(--primary-glow);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Sarabun', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
  line-height: 1.5;
  overflow-x: hidden;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: var(--bg-color);
  background-image: 
    radial-gradient(circle at 25% 25%, rgba(0, 255, 149, 0.05) 0%, transparent 50%),
    radial-gradient(circle at 75% 75%, rgba(255, 0, 200, 0.05) 0%, transparent 50%);
}

.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.header {
  background-color: rgba(10, 10, 10, 0.8);
  backdrop-filter: blur(10px);
  position: sticky;
  top: 0;
  z-index: 100;
  padding: 1rem 0;
  border-bottom: 1px solid var(--border-color);
  position: relative;
  overflow: hidden;
}

.gradient-animation {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color), var(--primary-color));
  background-size: 200% 100%;
  animation: gradient-slide 3s linear infinite;
}

@keyframes gradient-slide {
  0% { background-position: 0% 0; }
  100% { background-position: 200% 0; }
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  display: flex;
  flex-direction: column;
  line-height: 1;
}

.logo-text {
  font-size: 1.2rem;
  font-weight: 700;
  letter-spacing: 2px;
  color: var(--text-color);
}

.logo-accent {
  font-size: 1.8rem;
  font-weight: 800;
  letter-spacing: 1px;
}

.theme-button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.25rem;
  color: var(--text-color);
  transition: transform 0.3s;
}

.theme-button:hover {
  transform: rotate(30deg);
}

.main-content {
  flex: 1;
  padding: 2rem 0;
}

/* Glass Card Effect */
.glass-card {
  background: var(--glass-bg);
  border-radius: 16px;
  box-shadow: var(--shadow);
  border: 1px solid var(--glass-border);
  backdrop-filter: blur(8px);
  position: relative;
  overflow: hidden;
  transition: transform 0.3s, box-shadow 0.3s;
}

.glass-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px 0 rgba(0, 0, 0, 0.5);
}

.card-glow {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(
    circle at center,
    var(--primary-glow) 0%,
    transparent 20%
  );
  opacity: 0;
  transition: opacity 0.3s;
  pointer-events: none;
}

.glass-card:hover .card-glow {
  opacity: 0.1;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% { transform: scale(0.95); opacity: 0.1; }
  50% { transform: scale(1); opacity: 0.2; }
  100% { transform: scale(0.95); opacity: 0.1; }
}

.profile-section, .academic-section {
  margin-bottom: 2rem;
}

.profile-card {
  margin-bottom: 1.5rem;
}

.profile-header {
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  position: relative;
  border-bottom: 1px solid var(--border-color);
}

.profile-image-container {
  position: relative;
  width: 150px;
  height: 150px;
  border-radius: 50%;
  overflow: hidden;
  border: 3px solid var(--primary-color);
  box-shadow: 0 0 15px var(--primary-glow);
}

.profile-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s;
}

.profile-image-container:hover .profile-image {
  transform: scale(1.1);
}

.profile-image-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s;
}

.profile-image-container:hover .profile-image-overlay {
  opacity: 1;
}

.edit-photo-btn {
  background-color: var(--primary-color);
  color: black;
  border: none;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 1.2rem;
  transition: transform 0.3s;
}

.edit-photo-btn:hover {
  transform: scale(1.1);
}

.profile-info {
  text-align: center;
}

.student-name {
  font-size: 1.8rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.student-id {
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
  font-size: 1rem;
}

.student-major {
  font-weight: 500;
  margin-bottom: 0.75rem;
  font-size: 1.1rem;
}

.status-badge {
  display: inline-block;
  background: linear-gradient(90deg, var(--primary-color), var(--primary-hover));
  color: black;
  padding: 0.35rem 1rem;
  border-radius: 9999px;
  font-size: 0.9rem;
  font-weight: 600;
  box-shadow: 0 0 10px var(--primary-glow);
}

.neon-button {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.2rem;
  background-color: transparent;
  color: var(--primary-color);
  border: 1px solid var(--primary-color);
  border-radius: 4px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s;
  box-shadow: 0 0 5px var(--primary-glow);
}

.neon-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(0, 255, 149, 0.2),
    transparent
  );
  transition: left 0.7s;
}

.neon-button:hover {
  background-color: var(--primary-color);
  color: black;
  box-shadow: 0 0 15px var(--primary-glow);
}

.neon-button:hover::before {
  left: 100%;
}

.edit-profile-btn {
  margin-top: 0.5rem;
}

.profile-details {
  padding: 1.5rem 2rem;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  padding: 1rem 0;
  border-bottom: 1px solid var(--border-color);
}

.detail-item:last-child {
  border-bottom: none;
}

.detail-label {
  color: var(--text-secondary);
  font-weight: 500;
}

.detail-value {
  font-weight: 500;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.section-title {
  font-size: 1.5rem;
  font-weight: 700;
  position: relative;
  padding-left: 1rem;
}

.section-title::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 70%;
  background: linear-gradient(to bottom, var(--primary-color), var(--secondary-color));
  border-radius: 2px;
}

.add-course-btn {
  background: linear-gradient(90deg, var(--primary-color), var(--primary-hover));
  color: black;
}

.academic-summary {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.summary-item {
  padding: 1.5rem;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: transform 0.3s;
}

.summary-icon {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.summary-value {
  font-size: 2rem;
  font-weight: 700;
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 0.25rem;
}

.summary-label {
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.courses-table-container {
  padding: 1.5rem;
}

.courses-table {
  width: 100%;
  border-collapse: collapse;
}

.courses-table th,
.courses-table td {
  padding: 1rem;
  text-align: left;
  border-bottom: 1px solid var(--border-color);
}

.courses-table th {
  color: var(--text-secondary);
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.8rem;
  letter-spacing: 1px;
}

.course-row {
  transition: background-color 0.3s;
}

.course-row:hover {
  background-color: var(--card-bg-hover);
}

.grade {
  display: inline-block;
  padding: 0.35rem 0.75rem;
  border-radius: 4px;
  font-weight: 600;
  text-align: center;
  min-width: 40px;
}

.grade-good {
  background-color: rgba(0, 255, 149, 0.1);
  color: var(--primary-color);
  border: 1px solid var(--primary-color);
}

.grade-average {
  background-color: rgba(255, 204, 0, 0.1);
  color: var(--warning-color);
  border: 1px solid var(--warning-color);
}

.grade-warning {
  background-color: rgba(255, 153, 0, 0.1);
  color: #ff9900;
  border: 1px solid #ff9900;
}

.grade-fail {
  background-color: rgba(255, 51, 102, 0.1);
  color: var(--danger-color);
  border: 1px solid var(--danger-color);
}

.grade-other {
  background-color: rgba(160, 160, 160, 0.1);
  color: var(--text-secondary);
  border: 1px solid var(--text-secondary);
}

.actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 0.40rem;
}

.action-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.1rem;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.3s;
}

.edit-btn {
  color: var(--primary-color);
}

.edit-btn:hover {
  background-color: rgba(0, 255, 149, 0.1);
}

.delete-btn {
  color: var(--danger-color);
}

.delete-btn:hover {
  background-color: rgba(255, 51, 102, 0.1);
}

.carousel-section {
  margin-top: 3rem;
}

.carousel-container {
  position: relative;
  width: 100%;
  height: 300px;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: var(--shadow);
}

.carousel-slide {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 1s ease;
}

.carousel-slide.active {
  opacity: 1;
}

.carousel-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.carousel-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1.5rem;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8), transparent);
  color: white;
}

.carousel-caption h3 {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
}

.carousel-control {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  border-radius: 50%;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 10;
  transition: background-color 0.3s;
}

.carousel-control:hover {
  background-color: rgba(0, 0, 0, 0.8);
}

.carousel-control.prev {
  left: 10px;
}

.carousel-control.next {
  right: 10px;
}

.carousel-indicators {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  z-index: 10;
}

.indicator {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.5);
  border: none;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s;
}

.indicator.active {
  background-color: white;
  transform: scale(1.2);
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.glass-modal {
  background: var(--glass-bg);
  border-radius: 16px;
  box-shadow: var(--shadow);
  border: 1px solid var(--glass-border);
  backdrop-filter: blur(12px);
  width: 100%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s;
}

@keyframes slideUp {
  from { transform: translateY(50px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.confirmation-modal {
  max-width: 400px;
  text-align: center;
}

.modal-header {
  padding: 1.5rem;
  border-bottom: 1px solid var(--border-color);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h3 {
  font-size: 1.5rem;
  font-weight: 600;
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.8rem;
  cursor: pointer;
  color: var(--text-secondary);
  transition: color 0.3s;
}

.close-btn:hover {
  color: var(--text-color);
}

.modal-body {
  padding: 1.5rem;
}

.confirm-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
  color: var(--danger-color);
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: var(--text-secondary);
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background-color: rgba(0, 0, 0, 0.2);
  color: var(--text-color);
  font-size: 1rem;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 2px var(--primary-glow);
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
  margin-top: 2rem;
}

.cancel-btn,
.save-btn,
.delete-confirm-btn {
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.cancel-btn {
  background-color: transparent;
  border: 1px solid var(--border-color);
  color: var(--text-color);
}

.cancel-btn:hover {
  background-color: rgba(255, 255, 255, 0.05);
}

.save-btn {
  background-color: var(--primary-color);
  border: none;
  color: black;
}

.delete-confirm-btn {
  background-color: var(--danger-color);
  border: none;
  color: white;
  box-shadow: 0 0 10px rgba(255, 51, 102, 0.5);
}

.delete-confirm-btn:hover {
  background-color: #ff1a4d;
  box-shadow: 0 0 15px rgba(255, 51, 102, 0.7);
}

.photo-upload-container {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
}

.current-photo {
  width: 180px;
  height: 180px;
  border-radius: 50%;
  overflow: hidden;
  border: 3px solid var(--primary-color);
  box-shadow: 0 0 15px var(--primary-glow);
}

.current-photo img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  width: 100%;
}

input[type="file"] {
  display: none;
}

.upload-btn {
  width: 100%;
  max-width: 200px;
  text-align: center;
}

.upload-hint {
  font-size: 0.9rem;
  color: var(--text-secondary);
  text-align: center;
}

.sidebar {
  position: fixed;
  top: 0;
  left: -280px;
  width: 280px;
  height: 100vh;
  background-color: var(--card-bg);
  z-index: 1000;
  transition: left 0.3s ease;
  box-shadow: var(--shadow);
  display: flex;
  flex-direction: column;
}

.sidebar.active {
  left: 0;
}

.sidebar-header {
  padding: 1.5rem;
  border-bottom: 1px solid var(--border-color);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.profile {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.profile-img {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid var(--primary-color);
}

.profile-info p {
  margin: 0;
  font-weight: 600;
}

.profile-info .subtext {
  font-size: 0.8rem;
  color: var(--text-secondary);
}

.close-sidebar {
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 1.5rem;
  cursor: pointer;
}

.menu {
  list-style: none;
  padding: 1.5rem 0;
  margin: 0;
  flex-grow: 1;
  overflow-y: auto;
}

.menu-item {
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
  border-left: 3px solid transparent;
}

.menu-item:hover {
  background-color: var(--card-bg-hover);
}

.menu-item.active {
  background-color: rgba(0, 255, 149, 0.1);
  border-left: 3px solid var(--primary-color);
}

.menu-toggle {
  position: fixed;
  bottom: 2rem;
  left: 2rem;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: var(--primary-color);
  color: black;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  cursor: pointer;
  z-index: 999;
  box-shadow: 0 0 15px var(--primary-glow);
  transition: transform 0.3s;
}

.menu-toggle:hover {
  transform: scale(1.1);
}

@media (min-width: 768px) {
  .container {
    padding: 0 2rem;
  }
  
  .profile-header {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    padding: 2rem;
  }
  
  .profile-info {
    text-align: left;
    flex: 1;
    padding: 0 2rem;
  }
}

@media (max-width: 768px) {
  .academic-summary {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .summary-item:last-child {
    grid-column: span 2;
  }
  
  .courses-table th:nth-child(2),
  .courses-table td:nth-child(2) {
    display: none;
  }
}

@media (max-width: 576px) {
  .container {
    padding: 0 1rem;
  }
  
  .profile-image-container {
    width: 120px;
    height: 120px;
  }
  
  .student-name {
    font-size: 1.5rem;
  }
  
  .academic-summary {
    grid-template-columns: 1fr;
  }
  
  .summary-item {
    grid-column: span 1 !important;
  }
  
  .courses-table th:nth-child(3),
  .courses-table td:nth-child(3) {
    display: none;
  }
  
  .carousel-container {
    height: 200px;
  }
}
</style>