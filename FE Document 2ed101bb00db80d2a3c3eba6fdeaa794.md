# FE Document

Member(Superviser): @Hoang Anh Tu K17

# FRONTEND  DOCUMENT

- 1. Mục tiêu tài liệu
    
    Tài liệu này được tạo nhằm:
    
    - Giải thích **rõ ràng công nghệ FE đang sử dụng** (React nhưng là loại nào)
    - Trả lời câu hỏi **“vì sao chọn stack này”** theo góc nhìn kỹ thuật
    - Chuẩn hóa cách code FE để **toàn team đồng bộ**
    - Làm tài liệu **onboarding** cho các thành viên Frontend mới

---

- 2. Công nghệ Frontend được sử dụng (Technology Stack)
    
    ### 2.1. Tổng quan kiến trúc
    
    Hệ thống được xây dựng theo mô hình **Frontend – Backend tách biệt**:
    
    - **Frontend**: ReactJS – Single Page Application (SPA)
    - **Backend**: Spring Boot – RESTful API
    
    Frontend và Backend giao tiếp thông qua **HTTP/JSON**, frontend không render HTML phía server.
    
    ---
    
    ### 2.2. Stack Frontend chi tiết
    
    Frontend của hệ thống được xây dựng bằng **ReactJS thuần (React core – library)**, không phải meta-framework.
    
    Stack cụ thể:
    
    - **React core** – xây dựng UI component
    - **Vite** – build tool & dev server
    - **React Router DOM** – client-side routing
    - **Context API** – quản lý state dùng chung ở mức vừa
    - **UI Library**: chọn **1 trong các thư viện** (PrimeReact / MUI / AntD) tùy theo định hướng UI của project
    
    > **Lưu ý quan trọng:**
    > 
    > 
    > Hệ thống **không sử dụng meta-framework** (framework trên React) như **Next.js** hoặc **Remix**.
    > 
    > **Lý do:** backend Spring Boot đã xử lý toàn bộ business logic, hệ thống không yêu cầu SSR/SEO và việc dùng meta-framework sẽ làm kiến trúc phức tạp không cần thiết.
    > 
    
    ---
    
    ### 2.3. Stack Backend liên quan (Spring Boot)
    
    Backend sử dụng **Spring Boot** với các vai trò chính:
    
    - Cung cấp **RESTful API** cho frontend
    - Xử lý toàn bộ **business logic**
    - Authentication / Authorization
    - Kết nối và quản lý Database
    
    Frontend **chỉ đóng vai trò client**, không xử lý logic nghiệp vụ phức tạp.
    
    ---
    
    ### 2.4. Vì sao chọn React SPA + Spring Boot?
    
    Việc kết hợp **React SPA + Spring Boot** được lựa chọn có chủ đích, dựa trên các yếu tố sau:
    
    - Spring Boot đảm nhiệm toàn bộ xử lý nghiệp vụ và bảo mật
    - React tập trung vào **UI/UX và tương tác người dùng**
    - Không có yêu cầu **Server-Side Rendering (SSR)** trong phạm vi hệ thống
    - SEO không phải yếu tố quan trọng
    - Kiến trúc REST API của Spring Boot **phù hợp tự nhiên với React SPA**
    - Dễ chia task FE–BE và dễ onboarding
    
    Lựa chọn này giúp hệ thống **đơn giản** 
    
    ---
    
    ### 2.5. Lý do không sử dụng React Meta-framework (Next.js, Remix)
    
    Các meta-framework như **Next.js** hoặc **Remix** không được sử dụng trong hệ thống vì:
    
    - **Không cần SSR/SSG**: backend không render HTML và không có yêu cầu SEO
    - **Trùng vai trò với backend** khi đã có Spring Boot
    - **Tăng độ phức tạp kiến trúc** không cần thiết
    - **Khó onboarding** hơn cho thành viên mới
    - Lợi ích không tương xứng với scope hiện tại

---

# 3. Cấu trúc thư mục Frontend (Ví dụ minh hoạ theo domain Listing)

> **Lưu ý:** Cấu trúc dưới đây là **ví dụ minh hoạ cho domain Listing**, không đại diện cho toàn bộ hệ thống Frontend.
> 
> 
> Các domain khác (User, Auth, Admin, Transaction, …) sẽ áp dụng **cùng một tư duy tổ chức**, nhưng có cấu trúc và naming riêng theo từng Use Case.
> 

Toàn bộ naming trong ví dụ này tuân theo domain **Listing** (Marketplace), nhằm minh hoạ cách áp dụng domain-based naming trong frontend.

```
src/
 ├─ pages/
 │   ├─ ListingsPage.jsx          // Trang chính hiển thị Listings Feed
 │   ├─ ListingDetailPage.jsx     // Trang xem chi tiết một listing
 │   └─ CreateListingPage.jsx     // Trang tạo mới listing
 ├─ components/
 │   ├─ listing/
 │   │   ├─ ListingCard.jsx       // Component hiển thị thông tin tóm tắt của 1 listing
 │   │   └─ ListingsFeed.jsx      // Component render danh sách nhiều ListingCard
 ├─ services/
 │   └─ listingService.js         // Layer gọi API liên quan đến Listing
 ├─ hooks/
 │   └─ useListings.js            // Custom hook xử lý logic fetch & state listings
 ├─ routes/
 │   └─ AppRouter.jsx             // Khai báo toàn bộ client-side routes
 ├─ utils/                        // Hàm tiện ích dùng chung (format date, validate, ...)
 ├─ App.jsx                       // Root component, gắn router & layout chung
 └─ main.jsx                      // Entry point, render React app vào DOM

```

### 3.1 Giải thích chi tiết từng phần

- **pages/**: Chứa các component cấp trang (Page-level), mỗi file tương ứng với một route và một Use Case cụ thể.
    - `ListingsPage.jsx`: Trang hiển thị danh sách listing (Listings Feed), chịu trách nhiệm quản lý state ở mức page (loading, pagination, filter).
    - `ListingDetailPage.jsx`: Trang hiển thị chi tiết một listing, lấy dữ liệu dựa trên `listingId` từ URL.
    - `CreateListingPage.jsx`: Trang cho phép user tạo mới listing, xử lý form và submit dữ liệu.
- **components/**: Chứa các UI component tái sử dụng, không gắn trực tiếp với routing.
    - `ListingCard.jsx`: Hiển thị thông tin cơ bản của một listing (title, image, price, status).
    - `ListingsFeed.jsx`: Nhận danh sách listings từ page và render nhiều `ListingCard`.
- **services/**:
    - `listingService.js`: Chứa toàn bộ hàm gọi API liên quan đến Listing (get, create, update, hide), tách biệt khỏi UI.
- **hooks/**:
    - `useListings.js`: Custom hook gom logic gọi API, quản lý state listings và loading, giúp page component gọn nhẹ.
- **routes/**:
    - `AppRouter.jsx`: Khai báo các route của frontend, ánh xạ URL với page tương ứng.
- **utils/**:
    - Chứa các hàm tiện ích dùng chung trong frontend như format dữ liệu, validate input.
- **App.jsx**:
    - Root component, nơi gắn router và layout chung (header, footer nếu có).
- **main.jsx**:
    - Entry point của ứng dụng, render React app vào DOM.

**Ý nghĩa ví dụ:**

- Minh hoạ chi tiết cách tổ chức frontend theo **một domain cụ thể (Listing)**
- Giúp team hiểu rõ vai trò của từng thư mục và file
- Có thể áp dụng cùng tư duy cho các domain khác như User, Auth, Admin

### 3.2. Nguyên tắc tương tác giữa các package & System Design

Frontend được thiết kế theo mô hình:

> **Domain-based Structure + Layered Frontend Architecture**
> 

Mỗi domain tuân theo các layer rõ ràng, phân tách trách nhiệm.

### Các layer và vai trò

| Layer | Thư mục | Trách nhiệm |
| --- | --- | --- |
| Page | `pages/` | Đại diện cho màn hình, điều phối dữ liệu |
| UI Component | `components/` | Hiển thị UI |
| Logic | `hooks/` | Quản lý state, side-effect |
| Data Access | `services/` | Giao tiếp backend |
| Routing | `routes/` | Mapping URL |
| Utils | `utils/` | Hàm tiện ích |

---

# 4. Luồng hoạt động của một Feature (Feature Flow)

Ví dụ: **Listings Feed**

1. User truy cập `/listings`
2. `ListingsPage` được render
3. `useEffect` gọi `listingService.getListings()`
4. Backend Spring Boot trả JSON response
5. Data được lưu vào state
6. `ListingsFeed` và `ListingCard` render UI
7. User tương tác → update state → UI cập nhật

👉 Data flow **một chiều**, dễ debug và dễ đọc code.

---

# 5. State Management & Data Flow

### 5.1. Chiến lược quản lý state

Hệ thống **không sử dụng Redux/Zustand**.

Lý do:

- Quy mô hệ thống ở mức vừa
- State chủ yếu là UI state
- Business logic xử lý ở backend

State được quản lý bằng:

- **Local state**: `useState`
- **Side-effect**: `useEffect`
- **Global state** (khi cần): Context API

### 5.2. Data Flow giữa FE và BE

1. Component trigger action
2. Gọi function trong `listingService`
3. Gửi HTTP request đến Spring Boot
4. Nhận JSON response
5. Update state
6. Re-render UI

Frontend không xử lý nghiệp vụ, chỉ **hiển thị và gửi request**.

---

---

# 6. Quy ước code để đồng bộ team (Coding Conventions)

### 6.1. Component

- 1 file = 1 component
- Tên component dùng **PascalCase**
- Component chỉ xử lý **UI và user interaction**
- Component **không gọi API trực tiếp**

### 6.2. Service & Logic

- Tất cả API phải đi qua `services`
- Component **không chứa logic gọi API phức tạp**
- Logic liên quan đến dữ liệu, fetch, xử lý state nên được tách ra **custom hook**

### 6.3. Custom Hook

**Custom Hook** là một hàm JavaScript có tên bắt đầu bằng `use`, được dùng để **gom và tái sử dụng logic React** (state, side-effect, xử lý dữ liệu) giữa nhiều component.

Đặc điểm:

- Là function JavaScript
- Tên bắt đầu bằng `use`
- Có thể sử dụng các hook khác (`useState`, `useEffect`, `useContext`, ...)
- Không render UI

**Mục đích sử dụng custom hook:**

- Tách logic khỏi UI component
- Giữ component gọn, dễ đọc
- Tái sử dụng logic cho nhiều page/component
- Dễ test và dễ bảo trì

**Ví dụ trong hệ thống (Listings):**

```jsx
// hooks/useListings.js
import { useEffect, useState } from "react";
import { listingService } from "../services/listingService";

export function useListings() {
  const [listings, setListings] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    listingService.getListings()
      .then(res => setListings(res.data))
      .finally(() => setLoading(false));
  }, []);

  return { listings, loading };
}

```

Sử dụng trong component:

```jsx
function ListingsPage() {
  const { listings, loading } = useListings();

  return <ListingsFeed listings={listings} loading={loading} />;
}

```

👉 Component chỉ tập trung render UI, toàn bộ logic đã được tách sang custom hook.

### 6.4. Naming & Quy ước chung

- Áp dụng **domain-based naming** theo từng Use Case (ví dụ: Listing, User, Auth), đảm bảo các thành phần trong cùng domain được đặt tên nhất quán, tránh trộn lẫn giữa các domain
- Props đặt tên rõ nghĩa
- Không hard-code text (chuẩn bị cho i18n)
- Tránh duplicate logic

---

---

# 7. Executive Summary

**Frontend stack:**

- ReactJS (React core – SPA)
- Vite
- React Router DOM
- Context API
- UI Library (PrimeReact / MUI / AntD)

**Định hướng thiết kế:**

- Đơn giản
- Dễ mở rộng
- Dễ maintain
- Dễ đào tạo team

---

@Tran Thi Ngoc Anh (K18 HL) @La Thanh Hoa @Do Thanh An (K18 HL) @Le Duc Viet (K18 HL) 

## Video 1

**Cấu Trúc Project React.JS – Files Structure | React Cơ Bản Cho Beginners Từ A đến Z**

[https://www.youtube.com/watch?v=JsQfIjzaBNE](https://www.youtube.com/watch?v=JsQfIjzaBNE&utm_source=chatgpt.com)

### Video 2

**React Project Structure + Best Practices (React Core)**

🔗 [https://www.youtube.com/watch?v=5cCG4xUzABY](https://www.youtube.com/watch?v=5cCG4xUzABY&utm_source=chatgpt.com)

---

# 📘 HƯỚNG DẪN LÀM UI & FE TRONG TEAM

## React – Theo Cấu Trúc Project Chuẩn Của Team

---

## 1. Mục tiêu

Tài liệu này giúp member FE:

- Làm UI **đúng theo Figma**
- Code FE **đúng structure project của team**
- Biết **file nào viết cái gì**

---

# 2. Cấu trúc project FE của team (BẮT BUỘC TUÂN THỦ)

```
src/
 ├─ pages/
 │   ├─ ListingsPage.jsx
 │   ├─ ListingDetailPage.jsx
 │   └─ CreateListingPage.jsx
 ├─ components/
 │   └─ listing/
 │       ├─ ListingCard.jsx
 │       └─ ListingsFeed.jsx
 ├─ services/
 │   └─ listingService.js
 ├─ hooks/
 │   └─ useListings.js
 ├─ routes/
 │   └─ AppRouter.jsx
 ├─ utils/
 ├─ App.jsx
 └─ main.jsx
```

👉 **Mọi hướng dẫn bên dưới đều dựa trên cấu trúc này**

---

# 3. Quy trình làm UI & FE ()

```
Figma
 → pages (tạo page)
 → components (tách UI)
 → code UI tĩnh
 → hooks (logic + state)
 → services (API)
 → gắn vào page
```

---

# 📂 GIẢI THÍCH CHI TIẾT TỪNG THƯ MỤC

*(Theo đúng cấu trúc project của team)*

Cấu trúc frontend của team được thiết kế theo tư duy:

> **Page điều phối – Component hiển thị – Hook xử lý logic – Service gọi API**
> 

Mỗi thư mục **chỉ làm đúng 1 vai trò**, không chồng chéo.

## 1️⃣ `main.jsx` – Entry Point (KHÔNG ĐƯỢC ĐỘNG)

### Vai trò

- Là **điểm khởi động duy nhất** của app
- Render `<App />` vào DOM (`#root`)
- Khởi tạo các provider cấp cao (Router, Context, Theme nếu có)

Ví dụ tư duy:

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(
  <App />
);
```

### Quy ước bắt buộc

👉 **KHÔNG**:

- Viết UI
- Viết router
- Viết logic

👉 **KHÔNG tự ý chỉnh sửa** nếu không có yêu cầu từ leader FE

📌 Lý do:

Đây là file “xương sống”, sửa linh tinh dễ làm **toàn app lỗi**.

---

## 2️⃣ `App.jsx` – Root Component

### Vai trò

- Bao toàn bộ hệ thống
- Gắn router
- Gắn layout chung (Header / Footer / Sidebar)

Ví dụ tư duy:

```jsx
function App() {
  return (
    <BrowserRouter>
      <AppRouter />
    </BrowserRouter>
  );
}
```

### App.jsx **CHỈ làm**

- Gắn router
- Gắn layout tổng

### App.jsx **KHÔNG làm**

❌ Call API

❌ Xử lý business logic

❌ Fetch data

❌ Quản lý state phức tạp

📌 Lý do:

App.jsx càng đơn giản → app càng **dễ maintain & scale**.

---

## 3️⃣ `routes/AppRouter.jsx` – Điều hướng hệ thống

### Vai trò

- Mapping **URL → Page**
- Mỗi route trỏ tới **đúng 1 Page**

Ví dụ:

```jsx
<Route path="/listings" element={<ListingsPage />} />
<Route path="/listings/:id" element={<ListingDetailPage />} />
```

### Quy ước tư duy

- `route` → **page**
- **KHÔNG** route → component nhỏ

👉 Ví dụ đúng:

```
/listings → ListingsPage
```

👉 Ví dụ sai ❌:

```
/listings → ListingsFeed
```

📌 Lý do:

Page mới là nơi xử lý **flow + logic**, component chỉ là UI.

---

## 4️⃣ `pages/` – Page-level (TRUNG TÂM ĐIỀU PHỐI)

### Page là gì?

👉 Page đại diện cho **1 màn hình hoàn chỉnh**

👉 Page luôn gắn với **1 route**

### Page **CHỊU TRÁCH NHIỆM**

- Gọi custom hook
- Xử lý:
    - loading
    - error
    - empty state
- Truyền props xuống component

Ví dụ tư duy:

```jsx
function ListingsPage() {
  const { listings, loading } = useListings();

  if (loading) return <Loading />;

  return <ListingsFeed listings={listings} />;
}
```

### Page **KHÔNG ĐƯỢC**

❌ Call axios

❌ Viết UI chi tiết

❌ Viết logic fetch phức tạp

📌 Page = **orchestrator (điều phối)**, không phải nơi làm tất cả.

---

## 5️⃣ `components/` – UI Component (HIỂN THỊ)

### Component là gì?

- Là **UI nhỏ**
- Có thể reuse
- Không biết dữ liệu đến từ đâu

### Component **CHỈ làm**

- Nhận props
- Render UI
- Emit event (onClick, onChange)

Ví dụ:

```jsx
function ListingCard({ title, price }) {
  return (
    <div>
      <h3>{title}</h3>
      <p>{price}</p>
    </div>
  );
}
```

### Component **TUYỆT ĐỐI KHÔNG**

❌ Call API

❌ useEffect fetch data

❌ Xử lý business logic

📌 Lý do:

Component càng “ngu” → càng **dễ reuse & test**.

---

## 6️⃣ `hooks/` – Logic & State (BỘ NÃO)

### Hook dùng để làm gì?

- Gom logic
- Fetch data
- Quản lý state
- Xử lý side-effect

Ví dụ:

```jsx
export function useListings() {
  const [listings, setListings] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    listingService.getListings()
      .then(res => setListings(res.data))
      .finally(() => setLoading(false));
  }, []);

  return { listings, loading };
}
```

### Hook **CHỈ làm**

- Logic
- State
- Fetch

### Hook **KHÔNG**

❌ Render JSX

❌ Can thiệp UI

📌 Hook = **logic layer** của frontend.

---

## 7️⃣ `services/` – API Layer (CỔNG GIAO TIẾP BE)

### Service dùng để

- Gọi API
- Trả dữ liệu thô (promise)

Ví dụ:

```jsx
export const getListings = () => {
  return axios.get('/api/listings');
};
```

### Service **KHÔNG**

❌ Quản lý loading

❌ Set state

❌ Handle UI

📌 Service chỉ biết:

> “Gọi BE và trả dữ liệu về”
> 

---

## 8️⃣ `utils/` – Hàm dùng chung

### Dùng cho

- Format date
- Validate input
- Convert data

Ví dụ:

```jsx
export const formatPrice = (price) => `${price} VND`;
```

👉 **KHÔNG viết logic nghiệp vụ ở đây**

---

---

---

# 5. Chi tiết từng bước làm FE theo đúng structure

## 5.1 Bước 1 – Làm Page (`pages/`)

📌 Page chịu trách nhiệm:

- Lấy data (thông qua hook)
- Truyền props
- Render layout tổng

📌 Ví dụ: `ListingsPage.jsx`

```jsx
import { useListings } from "../hooks/useListings";
import ListingsFeed from "../components/listing/ListingsFeed";

function ListingsPage() {
  const { listings, loading } = useListings();

  if (loading) return <p>Loading...</p>;

  return <ListingsFeed listings={listings} />;
}

export default ListingsPage;
```

❌ Page **KHÔNG**:

- Call axios
- Render UI chi tiết từng item

---

## 5.2 Bước 2 – Làm Component UI (`components/`)

📌 Component chỉ:

- Nhận props
- Render UI

📌 `ListingCard.jsx`

```jsx
function ListingCard({ title, price, image }) {
  return (
    <div className="card">
      <img src={image} />
      <h3>{title}</h3>
      <p>{price}</p>
    </div>
  ****);
}

export default ListingCard;
```

📌 `ListingsFeed.jsx`

```jsx
import ListingCard from "./ListingCard";

function ListingsFeed({ listings }) {
  return (
    <div>
      {listings.map(item => (
        <ListingCard
          key={item.id}
          title={item.title}
          price={item.price}
          image={item.image}
        />
      ))}
    </div>
  );
}

export default ListingsFeed;
```

❌ Component **KHÔNG**:

- Fetch data
- Xử lý logic nghiệp vụ

---

## 5.3 Bước 3 – Logic & State (`hooks/`)

📌 Tất cả logic fetch data **BẮT BUỘC** nằm trong hook

📌 `useListings.js`

```jsx
import { useEffect, useState } from "react";
import { listingService } from "../services/listingService";

export function useListings() {
  const [listings, setListings] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    listingService.getListings()
      .then(res => setListings(res.data))
      .finally(() => setLoading(false));
  }, []);

  return { listings, loading };
}
```

👉 Page chỉ gọi hook, **không xử lý logic phức tạp**

---

## 5.4 Bước 4 – Call API (`services/`)

📌 `listingService.js`

```jsx
import axios from "axios";

export const listingService = {
  getListings() {
    return axios.get("/api/listings");
  },

  getListingDetail(id) {
    return axios.get(`/api/listings/${id}`);
  },

  createListing(data) {
    return axios.post("/api/listings", data);
  }
};
```

❌ Không gọi axios trực tiếp trong:

- Page
- Component

---

## 5.5 Bước 5 – Routing (`routes/`)

📌 `AppRouter.jsx`

```jsx
import { Routes, Route } from "react-router-dom";
import ListingsPage from "../pages/ListingsPage";
import ListingDetailPage from "../pages/ListingDetailPage";
import CreateListingPage from "../pages/CreateListingPage";

function AppRouter() {
  return (
    <Routes>
      <Route path="/listings" element={<ListingsPage />} />
      <Route path="/listings/:id" element={<ListingDetailPage />} />
      <Route path="/listings/create" element={<CreateListingPage />} />
    </Routes>
  );
}

export default AppRouter;
```

---

---

## 

# 6. Checklist bắt buộc trước khi merge code

✅ Page chỉ điều phối

✅ Component chỉ render UI

✅ Hook xử lý logic & state

✅ Service gọi API

✅ Đúng folder – đúng vai trò

---

# 7. Những lỗi CẤM trong team

❌ Viết axios trong component

❌ Nhét logic vào UI

❌ Page render UI chi tiết

❌ Tạo file không theo structure

❌ Copy code không hiểu

---

## 

---

## 

---

##