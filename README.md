# CookChella API Contract Documentation

## Overview

| Item | Detail |
|------|--------|
| **Base URL (Production)** | `https://cookchella-api.onrender.com/api/v1` |
| **Base URL (Development)** | `http://localhost:8000/api/v1` |
| **API Version** | v1 |
| **Response Format** | JSON |
| **Authentication** | Bearer Token (Laravel Sanctum) |

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [User Management](#2-user-management)
3. [Recipes](#3-recipes)
4. [Bookmarks](#4-bookmarks)
5. [Search](#5-search)
6. [Master Data](#6-master-data)
7. [Error Handling](#7-error-handling)
8. [Rate Limiting](#8-rate-limiting)

---

## Headers

### Request Headers

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}
X-Device-Platform: android|ios|web
X-App-Version: 1.0. 0
Accept-Language: id|en
```

### Response Headers

```http
Content-Type: application/json
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-Request-Id: uuid
```

---

## Standard Response Format

### Success Response

```json
{
  "success": true,
  "message": "Operation successful",
  "data": { },
  "meta": {
    "timestamp": "2025-11-27T10:00:00. 000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Paginated Response

```json
{
  "success": true,
  "message": "Data retrieved successfully",
  "data": [],
  "pagination": {
    "current_page": 1,
    "last_page": 10,
    "per_page": 10,
    "total": 100,
    "from": 1,
    "to": 10,
    "has_more_pages": true
  },
  "links": {
    "first": "https://api.example.com/v1/recipes?page=1",
    "last": "https://api.example.com/v1/recipes?page=10",
    "prev": null,
    "next": "https://api.example.com/v1/recipes?page=2"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Error Response

```json
{
  "success": false,
  "message": "Error message",
  "errors": {
    "field_name": ["Error detail 1", "Error detail 2"]
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

## 1. Authentication

### 1.1 Register

Mendaftarkan user baru dengan email dan password.

```http
POST /auth/register
```

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | string | Yes | min:2, max:100 |
| `username` | string | Yes | min:3, max:50, unique, alpha_dash |
| `email` | string | Yes | email, unique, max:255 |
| `password` | string | Yes | min:8, confirmed |
| `password_confirmation` | string | Yes | same:password |

```json
{
  "name": "John Doe",
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "password_confirmation": "SecurePass123!"
}
```

**Response Success (201 Created):**

```json
{
  "success": true,
  "message": "Registrasi berhasil",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "username": "johndoe",
      "email": "john@example. com",
      "avatar_url": null,
      "followers_count": 0,
      "following_count": 0,
      "recipes_count": 0,
      "created_at": "2025-11-27T10:00:00.000000Z"
    },
    "token": {
      "access_token": "1|laravel_sanctum_token_here.. .",
      "token_type": "Bearer",
      "expires_at": "2025-12-27T10:00:00.000000Z"
    }
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**Response Error (422 Unprocessable Entity):**

```json
{
  "success": false,
  "message": "Validasi gagal",
  "errors": {
    "email": ["Email sudah terdaftar"],
    "username": ["Username sudah digunakan"],
    "password": ["Password minimal 8 karakter"]
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 1.2 Login

Autentikasi user dengan email dan password.

```http
POST /auth/login
```

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `email` | string | Yes | email |
| `password` | string | Yes | min:1 |
| `device_name` | string | No | max:255 (untuk Sanctum token) |

```json
{
  "email": "john@example.com",
  "password": "SecurePass123!",
  "device_name": "Samsung Galaxy S21"
}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Login berhasil",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "username": "johndoe",
      "email": "john@example.com",
      "avatar_url": "https://xxxx. supabase.co/storage/v1/object/public/avatars/1/avatar.jpg",
      "followers_count": 100,
      "following_count": 50,
      "recipes_count": 25,
      "language": "id",
      "created_at": "2025-11-27T10:00:00.000000Z"
    },
    "token": {
      "access_token": "1|laravel_sanctum_token_here...",
      "token_type": "Bearer",
      "expires_at": "2025-12-27T10:00:00.000000Z"
    }
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**Response Error (401 Unauthorized):**

```json
{
  "success": false,
  "message": "Email atau password salah",
  "errors": null,
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 1.3 Login dengan Google (OAuth)

Login menggunakan Google OAuth.  Flutter akan mendapatkan ID Token dari Google Sign-In, kemudian mengirimkannya ke backend.

```http
POST /auth/google
```

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id_token` | string | Yes | Google ID Token dari Flutter |
| `device_name` | string | No | Nama device untuk token |

```json
{
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "device_name": "iPhone 14 Pro"
}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Login dengan Google berhasil",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "username": "johndoe_g123",
      "email": "john@gmail.com",
      "avatar_url": "https://lh3.googleusercontent.com/a/xxx",
      "followers_count": 0,
      "following_count": 0,
      "recipes_count": 0,
      "language": "id",
      "created_at": "2025-11-27T10:00:00.000000Z"
    },
    "token": {
      "access_token": "1|laravel_sanctum_token_here...",
      "token_type": "Bearer",
      "expires_at": "2025-12-27T10:00:00.000000Z"
    },
    "is_new_user": true
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00. 000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 1.4 Logout

Mencabut token akses user saat ini.

```http
POST /auth/logout
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Logout berhasil",
  "data": null,
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 1.5 Refresh Token

Mendapatkan token baru sebelum token lama expired.

```http
POST /auth/refresh
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Token berhasil diperbarui",
  "data": {
    "token": {
      "access_token": "2|new_laravel_sanctum_token_here...",
      "token_type": "Bearer",
      "expires_at": "2025-12-28T10:00:00.000000Z"
    }
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 1.6 Check Auth Status

Mengecek apakah token masih valid.

```http
GET /auth/check
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Token valid",
  "data": {
    "authenticated": true,
    "user_id": 1,
    "token_expires_at": "2025-12-27T10:00:00.000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

## 2. User Management

### 2.1 Get Current User Profile

Mendapatkan profil user yang sedang login.

```http
GET /user/profile
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Profil berhasil diambil",
  "data": {
    "id": 1,
    "name": "User PyTorch",
    "username": "userpytorch6",
    "email": "pytorch@example.com",
    "avatar_url": "https://xxxx.supabase.co/storage/v1/object/public/avatars/1/avatar. jpg",
    "followers_count": 100,
    "following_count": 50,
    "recipes_count": 25,
    "language": "id",
    "email_verified_at": "2025-11-27T10:00:00.000000Z",
    "created_at": "2025-11-27T10:00:00.000000Z",
    "updated_at": "2025-11-27T10:00:00.000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 2.2 Update Profile

Mengupdate profil user (name, username, avatar).

```http
PUT /user/profile
```

**Headers:**

```http
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request Body (multipart/form-data):**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | string | No | min:2, max:100 |
| `username` | string | No | min:3, max:50, unique, alpha_dash |
| `avatar` | file | No | image, mimes:jpeg,png,jpg,webp, max:2048 |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Profil berhasil diperbarui",
  "data": {
    "id": 1,
    "name": "New Name",
    "username": "newusername",
    "email": "pytorch@example.com",
    "avatar_url": "https://xxxx.supabase.co/storage/v1/object/public/avatars/1/avatar_1701234567.jpg",
    "followers_count": 100,
    "following_count": 50,
    "recipes_count": 25,
    "language": "id",
    "created_at": "2025-11-27T10:00:00.000000Z",
    "updated_at": "2025-11-27T12:00:00.000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T12:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 2.3 Update Language

Mengubah preferensi bahasa user.

```http
PUT /user/language
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `language` | string | Yes | in:id,en |

```json
{
  "language": "en"
}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Bahasa berhasil diperbarui",
  "data": {
    "language": "en"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 2.4 Get User by Username

Mendapatkan profil public user berdasarkan username. 

```http
GET /users/{username}
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | string | Username user |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Data user berhasil diambil",
  "data": {
    "id": 2,
    "name": "User 1",
    "username": "user1",
    "avatar_url": "https://xxxx.supabase.co/storage/v1/object/public/avatars/2/avatar.jpg",
    "followers_count": 100,
    "following_count": 50,
    "recipes_count": 25,
    "is_followed": false,
    "created_at": "2025-11-27T10:00:00. 000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 2.5 Follow User

Mengikuti user lain.

```http
POST /users/{username}/follow
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | string | Username user yang akan di-follow |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Berhasil mengikuti user",
  "data": {
    "is_followed": true,
    "followers_count": 101
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 2.6 Unfollow User

Berhenti mengikuti user. 

```http
DELETE /users/{username}/follow
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | string | Username user yang akan di-unfollow |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Berhasil berhenti mengikuti user",
  "data": {
    "is_followed": false,
    "followers_count": 100
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

## 3. Recipes

### 3.1 Get Timeline (Feed)

Mendapatkan feed timeline resep terbaru.

```http
GET /recipes/timeline
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Nomor halaman |
| `per_page` | integer | 10 | Jumlah item per halaman (max: 50) |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Timeline berhasil diambil",
  "data": [
    {
      "id": 1,
      "title": "Chicken Katsu ala Hokben",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
      "description": "Lorem ipsum dolor sit amet consectetur.. .",
      "cooking_time": 60,
      "servings": 3,
      "likes_count": 2500,
      "bookmarks_count": 150,
      "is_liked": false,
      "is_bookmarked": true,
      "user": {
        "id": 1,
        "name": "User 1",
        "username": "user1",
        "avatar_url": "https://xxxx. supabase.co/storage/v1/object/public/avatars/1/avatar.jpg"
      },
      "created_at": "2025-11-27T10:00:00. 000000Z"
    },
    {
      "id": 2,
      "title": "Ayam Goreng Mentega",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/2/image.jpg",
      "description": "Resep ayam goreng mentega yang lezat...",
      "cooking_time": 45,
      "servings": 4,
      "likes_count": 1800,
      "bookmarks_count": 120,
      "is_liked": true,
      "is_bookmarked": false,
      "user": {
        "id": 2,
        "name": "User 2",
        "username": "user2",
        "avatar_url": null
      },
      "created_at": "2025-11-26T10:00:00.000000Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "last_page": 10,
    "per_page": 10,
    "total": 100,
    "from": 1,
    "to": 10,
    "has_more_pages": true
  },
  "links": {
    "first": "https://cookchella-api.onrender.com/api/v1/recipes/timeline?page=1",
    "last": "https://cookchella-api.onrender.com/api/v1/recipes/timeline?page=10",
    "prev": null,
    "next": "https://cookchella-api.onrender.com/api/v1/recipes/timeline?page=2"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3. 2 Get Recipe Recommendations

Mendapatkan rekomendasi resep untuk section "Coba masak ini". 

```http
GET /recipes/recommendations
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 5 | Jumlah rekomendasi (max: 10) |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Rekomendasi resep berhasil diambil",
  "data": [
    {
      "id": 1,
      "title": "Chicken Katsu ala Hokben",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
      "cooking_time": 60,
      "likes_count": 2500
    },
    {
      "id": 3,
      "title": "Pisang Goreng Madu",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/3/image.jpg",
      "cooking_time": 30,
      "likes_count": 1500
    }
  ],
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3.3 Get Recipe Detail

Mendapatkan detail lengkap resep.

```http
GET /recipes/{id}
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token} (optional untuk guest)
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Detail resep berhasil diambil",
  "data": {
    "id": 1,
    "title": "Chicken Katsu ala Hokben",
    "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
    "description": "Lorem ipsum dolor sit amet consectetur.  Felis ac mi scelerisque non.  Tempus fermentum sed pharetra ac rhoncus cursus erat lectus sodales.",
    "cooking_time": 60,
    "servings": 3,
    "likes_count": 2500,
    "bookmarks_count": 150,
    "is_liked": false,
    "is_bookmarked": true,
    "user": {
      "id": 1,
      "name": "User 1",
      "username": "user1",
      "avatar_url": "https://xxxx.supabase.co/storage/v1/object/public/avatars/1/avatar.jpg",
      "followers_count": 100,
      "is_followed": false
    },
    "ingredients": [
      {
        "id": 1,
        "name": "Dada Ayam",
        "quantity": "500",
        "unit": "gram"
      },
      {
        "id": 2,
        "name": "Tepung Panir",
        "quantity": "200",
        "unit": "gram"
      },
      {
        "id": 3,
        "name": "Telur",
        "quantity": "2",
        "unit": "butir"
      },
      {
        "id": 4,
        "name": "Garam",
        "quantity": "1",
        "unit": "sdt"
      }
    ],
    "steps": [
      {
        "id": 1,
        "step_number": 1,
        "description": "Potong dada ayam menjadi lembaran tipis",
        "image_url": null
      },
      {
        "id": 2,
        "step_number": 2,
        "description": "Kocok telur dalam mangkuk",
        "image_url": null
      },
      {
        "id": 3,
        "step_number": 3,
        "description": "Celupkan ayam ke telur lalu gulingkan ke tepung panir",
        "image_url": null
      },
      {
        "id": 4,
        "step_number": 4,
        "description": "Goreng dalam minyak panas hingga golden brown",
        "image_url": null
      }
    ],
    "created_at": "2025-11-27T10:00:00. 000000Z",
    "updated_at": "2025-11-27T10:00:00.000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3. 4 Create Recipe

Membuat resep baru.

```http
POST /recipes
```

**Headers:**

```http
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request Body (multipart/form-data):**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | Yes | min:3, max:255 |
| `image` | file | Yes | image, mimes:jpeg,png,jpg,webp, max:5120 |
| `description` | string | No | max:1000 |
| `cooking_time` | integer | No | min:1 (dalam menit) |
| `servings` | integer | No | min:1 |
| `ingredients` | array | Yes | min:1 |
| `ingredients[*][name]` | string | Yes | max:100 |
| `ingredients[*][quantity]` | string | Yes | max:50 |
| `ingredients[*][unit]` | string | Yes | max:50 |
| `steps` | array | Yes | min:1 |
| `steps[*][description]` | string | Yes | max:500 |

**Example Request (Flutter - Dio):**

```dart
final formData = FormData.fromMap({
  'title': 'Chicken Katsu ala Hokben',
  'image': await MultipartFile.fromFile(imagePath, filename: 'recipe.jpg'),
  'description': 'Resep chicken katsu yang enak dan mudah dibuat',
  'cooking_time': 60,
  'servings': 3,
  'ingredients[0][name]': 'Dada Ayam',
  'ingredients[0][quantity]': '500',
  'ingredients[0][unit]': 'gram',
  'ingredients[1][name]': 'Tepung Panir',
  'ingredients[1][quantity]': '200',
  'ingredients[1][unit]': 'gram',
  'steps[0][description]': 'Potong dada ayam menjadi lembaran tipis',
  'steps[1][description]': 'Lumuri dengan tepung panir',
});
```

**Response Success (201 Created):**

```json
{
  "success": true,
  "message": "Resep berhasil dibagikan",
  "data": {
    "id": 10,
    "title": "Chicken Katsu ala Hokben",
    "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/10/image_1701234567.jpg",
    "description": "Resep chicken katsu yang enak dan mudah dibuat",
    "cooking_time": 60,
    "servings": 3,
    "likes_count": 0,
    "bookmarks_count": 0,
    "is_liked": false,
    "is_bookmarked": false,
    "user": {
      "id": 1,
      "name": "User 1",
      "username": "user1",
      "avatar_url": "https://xxxx.supabase.co/storage/v1/object/public/avatars/1/avatar.jpg"
    },
    "ingredients": [
      {
        "id": 1,
        "name": "Dada Ayam",
        "quantity": "500",
        "unit": "gram"
      },
      {
        "id": 2,
        "name": "Tepung Panir",
        "quantity": "200",
        "unit": "gram"
      }
    ],
    "steps": [
      {
        "id": 1,
        "step_number": 1,
        "description": "Potong dada ayam menjadi lembaran tipis",
        "image_url": null
      },
      {
        "id": 2,
        "step_number": 2,
        "description": "Lumuri dengan tepung panir",
        "image_url": null
      }
    ],
    "created_at": "2025-11-27T12:00:00.000000Z",
    "updated_at": "2025-11-27T12:00:00.000000Z"
  },
  "meta": {
    "timestamp": "2025-11-27T12:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3. 5 Update Recipe

Mengupdate resep yang sudah ada.  Hanya pemilik resep yang dapat mengupdate.

```http
PUT /recipes/{id}
POST /recipes/{id}? _method=PUT
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request Body (multipart/form-data):**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | No | min:3, max:255 |
| `image` | file | No | image, mimes:jpeg,png,jpg,webp, max:5120 |
| `description` | string | No | max:1000 |
| `cooking_time` | integer | No | min:1 |
| `servings` | integer | No | min:1 |
| `ingredients` | array | No | min:1 (akan replace semua ingredients) |
| `steps` | array | No | min:1 (akan replace semua steps) |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Resep berhasil diperbarui",
  "data": {
    "id": 10,
    "title": "Updated Chicken Katsu",
    "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/10/image_1701234999.jpg",
    ... 
  },
  "meta": {
    "timestamp": "2025-11-27T14:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3.6 Delete Recipe

Menghapus resep.  Hanya pemilik resep yang dapat menghapus. 

```http
DELETE /recipes/{id}
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Resep berhasil dihapus",
  "data": null,
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3.7 Like Recipe

Memberikan like pada resep. 

```http
POST /recipes/{id}/like
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Resep berhasil disukai",
  "data": {
    "is_liked": true,
    "likes_count": 2501
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3.8 Unlike Recipe

Menghapus like dari resep.

```http
DELETE /recipes/{id}/like
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Like berhasil dihapus",
  "data": {
    "is_liked": false,
    "likes_count": 2500
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 3.9 Get User Recipes

Mendapatkan daftar resep milik user tertentu.

```http
GET /users/{username}/recipes
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | string | Username user |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Nomor halaman |
| `per_page` | integer | 10 | Jumlah item per halaman |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Daftar resep berhasil diambil",
  "data": [
    {
      "id": 1,
      "title": "Chicken Katsu ala Hokben",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
      "cooking_time": 60,
      "servings": 3,
      "likes_count": 2500,
      "created_at": "2025-11-27T10:00:00.000000Z"
    }
  ],
  "pagination": {... },
  "links": {...},
  "meta": {... }
}
```

---

## 4. Bookmarks

### 4.1 Get User Bookmarks

Mendapatkan daftar resep yang di-bookmark oleh user. 

```http
GET /bookmarks
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Nomor halaman |
| `per_page` | integer | 10 | Jumlah item per halaman |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Daftar bookmark berhasil diambil",
  "data": [
    {
      "id": 1,
      "bookmarked_at": "2025-11-27T10:00:00.000000Z",
      "recipe": {
        "id": 1,
        "title": "Chicken Katsu ala Hokben",
        "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
        "cooking_time": 60,
        "servings": 3,
        "likes_count": 2500,
        "ingredients": [
          {
            "id": 1,
            "name": "Dada Ayam",
            "quantity": "500",
            "unit": "gram"
          }
        ],
        "steps": [
          {
            "id": 1,
            "step_number": 1,
            "description": "Masukkan lorem ke dalam wajan"
          }
        ],
        "user": {
          "id": 1,
          "name": "User 1",
          "username": "user1",
          "avatar_url": "..."
        }
      }
    }
  ],
  "pagination": {
    "current_page": 1,
    "last_page": 5,
    "per_page": 10,
    "total": 45,
    "from": 1,
    "to": 10,
    "has_more_pages": true
  },
  "links": {... },
  "meta": {...}
}
```

---

### 4.2 Add Bookmark

Menambahkan resep ke bookmark.

```http
POST /recipes/{id}/bookmark
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (201 Created):**

```json
{
  "success": true,
  "message": "Resep berhasil ditambahkan ke bookmark",
  "data": {
    "is_bookmarked": true,
    "bookmarks_count": 151
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

### 4.3 Remove Bookmark

Menghapus resep dari bookmark.

```http
DELETE /recipes/{id}/bookmark
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | ID resep |

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Resep berhasil dihapus dari bookmark",
  "data": {
    "is_bookmarked": false,
    "bookmarks_count": 150
  },
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

---

## 5. Search

### 5.1 Search Recipes

Mencari resep berdasarkan keyword dan filter bahan.

```http
GET /search
```

**Headers:**

```http
Authorization: Bearer {access_token} (optional)
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | null | Keyword pencarian |
| `ingredients[]` | array | null | Array ID master ingredients untuk filter |
| `category` | string | null | Slug kategori (protein, bumbu_rempah, sayur, produk_olahan) |
| `cooking_time_max` | integer | null | Filter waktu masak maksimal (menit) |
| `sort_by` | string | latest | Sorting: latest, popular, cooking_time |
| `sort_order` | string | desc | asc atau desc |
| `page` | integer | 1 | Nomor halaman |
| `per_page` | integer | 10 | Jumlah item per halaman |

**Example Request:**

```http
GET /search? q=ayam&ingredients[]=1&ingredients[]=3&sort_by=popular&page=1
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Hasil pencarian",
  "data": [
    {
      "id": 1,
      "title": "Ayam Goreng Mentega",
      "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg",
      "description": "Resep ayam goreng mentega yang lezat.. .",
      "cooking_time": 45,
      "servings": 4,
      "likes_count": 2500,
      "is_liked": true,
      "is_bookmarked": false,
      "user": {
        "id": 1,
        "name": "User 1",
        "username": "user1",
        "avatar_url": "..."
      },
      "created_at": "2025-11-27T10:00:00.000000Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "last_page": 3,
    "per_page": 10,
    "total": 25,
    "from": 1,
    "to": 10,
    "has_more_pages": true
  },
  "filters_applied": {
    "query": "ayam",
    "ingredients": [1, 3],
    "category": null,
    "sort_by": "popular",
    "sort_order": "desc"
  },
  "links": {...},
  "meta": {...}
}
```

---

### 5.2 Get Filter Options

Mendapatkan opsi filter bahan (master ingredients) yang tersedia.

```http
GET /search/filters
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Filter options berhasil diambil",
  "data": {
    "categories": [
      {
        "id": 1,
        "name": "Protein",
        "slug": "protein",
        "ingredients": [
          {"id": 1, "name": "Sapi", "slug": "sapi"},
          {"id": 2, "name": "Ayam", "slug": "ayam"},
          {"id": 3, "name": "Ikan", "slug": "ikan"},
          {"id": 4, "name": "Udang", "slug": "udang"},
          {"id": 5, "name": "Telur", "slug": "telur"},
          {"id": 6, "name": "Tahu", "slug": "tahu"},
          {"id": 7, "name": "Tempe", "slug": "tempe"}
        ]
      },
      {
        "id": 2,
        "name": "Bumbu & Rempah",
        "slug": "bumbu_rempah",
        "ingredients": [
          {"id": 8, "name": "Cabe", "slug": "cabe"},
          {"id": 9, "name": "Kayu Manis", "slug": "kayu_manis"},
          {"id": 10, "name": "Bawang Merah", "slug": "bawang_merah"},
          {"id": 11, "name": "Bawang Putih", "slug": "bawang_putih"}
        ]
      },
      {
        "id": 3,
        "name": "Sayur",
        "slug": "sayur",
        "ingredients": [
          {"id": 12, "name": "Wortel", "slug": "wortel"},
          {"id": 13, "name": "Bayam", "slug": "bayam"},
          {"id": 14, "name": "Terong", "slug": "terong"},
          {"id": 15, "name": "Sawi", "slug": "sawi"}
        ]
      },
      {
        "id": 4,
        "name": "Produk Olahan",
        "slug": "produk_olahan",
        "ingredients": [
          {"id": 16, "name": "Sosis", "slug": "sosis"},
          {"id": 17, "name": "Bakso", "slug": "bakso"}
        ]
      }
    ]
  },
  "meta": {... }
}
```

---

### 5.3 Get Search History

Mendapatkan riwayat pencarian user.

```http
GET /search/history
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 10 | Jumlah history (max: 20) |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Riwayat pencarian berhasil diambil",
  "data": [
    {
      "id": 1,
      "keyword": "Ayam Goreng Mentega",
      "recipe": {
        "id": 2,
        "title": "Ayam Goreng Mentega",
        "image_url": "https://xxxx.supabase.co/storage/v1/object/public/recipes/2/image.jpg"
      },
      "searched_at": "2025-11-27T10:00:00.000000Z"
    },
    {
      "id": 2,
      "keyword": "Ayam Goreng Asam Manis",
      "recipe": null,
      "searched_at": "2025-10-28T10:00:00.000000Z"
    }
  ],
  "meta": {...}
}
```

---

### 5.4 Clear Search History

Menghapus semua riwayat pencarian user.

```http
DELETE /search/history
```

**Headers:**

```http
Authorization: Bearer {access_token}
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Riwayat pencarian berhasil dihapus",
  "data": null,
  "meta": {... }
}
```

---

## 6. Master Data

### 6.1 Get Ingredient Categories

Mendapatkan daftar kategori bahan. 

```http
GET /master/ingredient-categories
```

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Kategori bahan berhasil diambil",
  "data": [
    {
      "id": 1,
      "name": "Protein",
      "slug": "protein"
    },
    {
      "id": 2,
      "name": "Bumbu & Rempah",
      "slug": "bumbu_rempah"
    },
    {
      "id": 3,
      "name": "Sayur",
      "slug": "sayur"
    },
    {
      "id": 4,
      "name": "Produk Olahan",
      "slug": "produk_olahan"
    }
  ],
  "meta": {...}
}
```

---

### 6.2 Get Master Ingredients

Mendapatkan daftar master ingredients. 

```http
GET /master/ingredients
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `category_id` | integer | null | Filter by category ID |
| `category_slug` | string | null | Filter by category slug |

**Response Success (200 OK):**

```json
{
  "success": true,
  "message": "Master ingredients berhasil diambil",
  "data": [
    {
      "id": 1,
      "name": "Sapi",
      "slug": "sapi",
      "category": {
        "id": 1,
        "name": "Protein",
        "slug": "protein"
      }
    }
  ],
  "meta": {...}
}
```

---

## 7. Error Handling

### HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| `200` | OK - Request berhasil |
| `201` | Created - Resource berhasil dibuat |
| `204` | No Content - Request berhasil tanpa response body |
| `400` | Bad Request - Request tidak valid |
| `401` | Unauthorized - Token tidak valid atau tidak ada |
| `403` | Forbidden - Tidak memiliki akses |
| `404` | Not Found - Resource tidak ditemukan |
| `422` | Unprocessable Entity - Validasi gagal |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error - Error server |

### Error Response Examples

**401 Unauthorized:**

```json
{
  "success": false,
  "message": "Unauthenticated",
  "errors": null,
  "meta": {
    "timestamp": "2025-11-27T10:00:00.000000Z",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**403 Forbidden:**

```json
{
  "success": false,
  "message": "Anda tidak memiliki akses untuk melakukan aksi ini",
  "errors": null,
  "meta": {... }
}
```

**404 Not Found:**

```json
{
  "success": false,
  "message": "Resep tidak ditemukan",
  "errors": null,
  "meta": {...}
}
```

**422 Validation Error:**

```json
{
  "success": false,
  "message": "Validasi gagal",
  "errors": {
    "title": ["Nama resep wajib diisi"],
    "image": ["Foto resep wajib diupload", "Ukuran maksimal 5MB"],
    "ingredients": ["Minimal 1 bahan diperlukan"]
  },
  "meta": {...}
}
```

**429 Rate Limit:**

```json
{
  "success": false,
  "message": "Terlalu banyak request.  Silakan coba lagi dalam 60 detik.",
  "errors": {
    "retry_after": 60
  },
  "meta": {...}
}
```

---

## 8. Rate Limiting

| Endpoint Type | Limit | Window |
|--------------|-------|--------|
| Auth endpoints | 10 requests | per minute |
| General endpoints | 60 requests | per minute |
| Search endpoints | 30 requests | per minute |
| Upload endpoints | 10 requests | per minute |

**Rate Limit Headers:**

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1701234567
```

---

## Appendix: Supabase Storage URLs

### URL Patterns

| Type | URL Pattern |
|------|-------------|
| Avatar | `https://{project}.supabase.co/storage/v1/object/public/avatars/{user_id}/{filename}` |
| Recipe Image | `https://{project}.supabase.co/storage/v1/object/public/recipes/{recipe_id}/{filename}` |

### Image Transformations (via Supabase)

```
// Resize
? width=300&height=300

// Quality
?quality=80

// Format
?format=webp
```

**Example:**
```
https://xxxx.supabase.co/storage/v1/object/public/recipes/1/image.jpg?width=300&height=300&quality=80
```

