# 測試開發以 Laravel API 為例

# 概述

逐步開發一個 API 包括撰寫單元測試 (Unit Test) 是良好的開發實踐。以下是一個可能的步驟，以新增使用者為例：

# 前提

## 軟體設計架構 Controller-Service-Repository-Model

- **Controller（控制器）**：接收參數、派工給 Service、收到結果、處理輸出。Ex. 新增會員：接收資料，呼叫新增會員流程 Service 後取得結果，寫入 API log，然後把會員 ID 以 http status 200 用 JSON Object 輸出
- **Service（服務）**：收到資料、處理流程、資料處理、錯誤處理、輸出回應。Ex. 新增會員流程中，判斷是否已存在，建立會員，建立會員關聯資料，寫入 log，取得回傳值，並依狀況給予相對應的回應然後輸出結果。
- **Repository（儲存庫）**：以 Eloquent ORM 來實作 Query 內容、執行及取得、錯誤處理、輸出結果。Ex. 建立會員實作並回傳結果，基本上 1 個 DB 行為 1 個函式，依需求可調整。
- **Model（模型）**：設定各欄位特性、建立關聯。Ex. 設定 Users table, Primary key = ‘id’, timestamp = [’created_at’, ‘updated_at’]…

### 優點

- **組織清晰：** 這種結構使應用程式的不同層次清晰可見，提高了代碼的可讀性和可維護性。
- **可測試性：** 服務和儲存庫的抽象層使單元測試更加容易進行，減少了對外部資源（如資料庫）的依賴。
- **可擴展性：** 由於各個層次之間的解耦，更容易擴展和修改應用程式。

### **缺點**

- **增加複雜性：** 對於小型或簡單的應用程式，CSRM 結構可能會被認為過於繁複。
- **學習成本：** 新手可能需要一些時間來理解並適應這種結構，特別是對於初次使用 Laravel 的開發者來說。

總體而言，Controller-Service-Repository-Model 結構提供了清晰的組織和易於擴展的架構。

---

# 開發流程

1. 建立測試通道
2. API 功能開發
3. 進階測試 (option)

![image](https://github.com/redteaman/LaravelDevelop/blob/main/%E6%96%87%E4%BB%B6/1.drawio.png)

# 建立**測試通道**

## 建立測試

- 創建一個新的測試程鄉對應此次的 API 開發。
    
    ```bash
    $ php artisan make:test UserCreateApiTest
    ```
    
- 在測試檔案中，使用 Laravel 提供的 TestCase 類別來撰寫測試方法。
    
    **tests\Feature\UserCreateApiTest.php**
    
    ```php
    <?php
    namespace Tests\Feature;
    use Tests\TestCase;
    
    class UserCreateApiTest extends TestCase
    {
        // 測試新增
        public function testUserCreate()
        {
            // Your test logic for creating a user
        }   
    }
    ```
    

## 建立程式

- 在 **`app/Http/Controllers/API`** 目錄下創建 **`UsersController.php`**。
    
    ```bash
    $ php artisan make:controller UsersController --api
    ```
    
- 在控制器中實作相對應的新增函式。
    
    **App\Http\Controllers\API\UsersController**
    
    ```php
    <?php
    namespace App\Http\Controllers\API;
    use Illuminate\Http\Request;
    
    class UsersController extends Controller
    {
        public function create(Request $request)
        {
            // Your logic for creating a user
            return response()->json([
                'return' => true,
                'code' => 0000,
                'message' => 'OK',
                'data' => ['id' => 123],
            ], 200);
        }
    }
    ```
    

## 路由設定

- 將 API 端點對應到相對應的控制器方法。
    
    **route/api.php**
    
    ```php
    use App\Http\Controllers\UsersController;
    ...
    Route::post('/users', [UsersController::class, 'create']);
    ...
    ```
    

## 建立過濾

1. 為了過濾進入 API 的參數內容，建立 1 個 request 來保護 API 及流程。
    
    ```php
    $ php artisan make:request UserCreateRequest 
    ```
    
2. 依照文件給的規格及範例，將欄位格式的限制
    
    **App\Http\Requests\UserCreateRequest**
    
    ```php
    // app/Http/Requests/UserCreateRequest.php
    namespace App\Http\Requests;
    use Illuminate\Foundation\Http\FormRequest;
    
    class UserCreateRequest extends FormRequest
    {
        public function rules()
        {
            return [
                'name' => 'required|string',
                'work_id' => 'required|string',
                'is_active' => 'required|boolean',
                // 其他欄位的驗證規則
            ];
        }
    
        public function messages():array
        {
            return [
                "work_id.required" => "缺少 工號",
                "work_id.string" => "請輸入 工號",
                "name.required" => "缺少 姓名",
                "name.max" => "姓名 最多輸入 100 個字",
                "is_active.required" => "缺少 是否啟用",
                "is_active.boolean" => "請選擇 是否啟用",
            ];
        }
    }
    ```
    
3. 放到 Controller
    
    **App\Http\Controllers\API\UsersController**
    
    ```php
    **<?php**
    namespace App\Http\Controllers\API;
    use Illuminate\Http\Request;
    
    class UsersController extends Controller
    {
    ...
        public function create(UserRequest $request)
        {
    					// Your logic for creating a user
            return response()->json([
                'return' => true,
                'code' => 0000,
                'message' => 'OK',
                'data' => ['id' => 123],
            ], 200);
    		}
    ...
    }
    ```
    

## 撰寫基本測試規則

1. 依照狀況，將想測試的資料欄位寫進測試
    
    **tests\Feature\UserCreateApiTest.php**
    
    ```php
    <?php
    namespace Tests\Feature;
    use Tests\TestCase;
    
    class UserCreateApiTest extends TestCase
    {
        // 測試新增
        public function test_first_data()
        {
            // Your test logic for creating a member
            $api_url = '/api/users';
            $api_method = "POST";
    
            $data = [
                "name"=> "John Doe",
                "work_id"=> '0003322',
                "is_active"=> 1,
            ];
    
            $response = $this
                ->json($api_method, $api_url, $data);
    
            $response->assertStatus(200)
                ->assertJsonStructure([
                    'result',
                    'code',
                    'message',
                    'data',
                ]);
        }   
    }
    ```
    
2. 執行測試
    
    ```bash
    $ php artisan test --filter UserCreateApiTest
    ```
    
    如果可以看到綠燈，那測試通道就 OK 了
    

# API 功能開發

## 介面處理-Controller

- 將流程寫入，並且處理從 Service 帶回的狀態及資料，然後以 JSON 輸出
    
    **App\Http\Controllers\UsersController.php**
    
    ```php
    namespace App\Http\Controllers\API;
    use Illuminate\Http\JsonResponse;
    use App\Http\Requests\UserCreateRequest;
    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use App\Http\Services\UsersService;
    class UsersController extends Controller
    {
        protected UsersService $users;
    
        public function __construct( UsersService $users ) {
            $this->users = $users;
        }
    ...
        public function store(UserCreateRequest $request):JsonResponse
        {
            $service_data = $this->users->createItem($request->input());
    
            $response_code = $service_data['code'];
            $message = $service_data['message'];
            $http_status = ($response_code==="0000") ? 200: 400;
            $result = $response_code==="0000";
            // Your logic for creating a user
            return response()->json([
                'result' => $result,
                'code' => $response_code,
                'message' => $message,
                'data' => $service_data,
            ], $http_status);
        }
    ...
    }
    ```
    

## 流程開發-Service

- 整理好取得的資料，然後進行指定的流程操作
    
    **App\Http\Services\UsersService.php**
    
    ```php
    <?php
    namespace App\Http\Services;
    use App\Http\Repositories\UsersRepository;
    
    class UsersService
    {
        protected UsersRepository $users;
    		private array $return_array;
    
    		public function __construct(UsersRepository $users)
        {
            $this->users = $users;
    		}
    ...
        /**
         * @param array $input
         * @return array
         */
        public function createItem(array $input): array
        {
            try {
                $result = $this->users->createItem($input);
    
                if (!$result) {
                    throw new \Exception("0001");
                }
    
                $id = $result;
                $this->return_array = [
                    'result' => true,
                    'code' => "0000",
                    'id' => $id,
                    'message' => "OK"
                ];
            } catch (\Exception $e) {
                $this->return_array = [
                    'result' => false,
                    'code' => 0001,
                    'id' => null,
                    'message' => $e->getMessage()
                ];
            }
            return $this->return_array;
        }
    ...
    }
    ```
    

## 資料存取

- 將要執行的資料存取的行為或 Query 用 eloquent 來執行並回傳結果
    
    **App\Http\Repositories\UsersRepository.php**
    
    ```php
    <?php
    
    namespace App\Http\Repositories;
    
    use Illuminate\Support\Facades\DB;
    use App\Models\Users;
    
    class UsersRepository
    {
    ...
        public function createItem(array $input): mixed
        {
            try {
                $user = Users::create($input);
                return $user->id;
            } catch (\Exception $e) {
                // 可以在這裡處理例外，例如寫入 Log 或其他操作
                return false;
            }
        }
    ...
    }
    ```
    

## 測試 API

- 以之前的基本測試腳本來測試結果，並與 DB 對照
    
    ```bash
    $ php artisan test
    ```
    

# 進階測試

1. 建立假資料 (使用 Factory)
    
    ```bash
    $ php artisan make:factory UsersFactory
    ```
    
2. 設計假資料
    
    **Database\Factories\UsersFactory.php**
    
    ```php
    <?php
    namespace Database\Factories;
    use Illuminate\Database\Eloquent\Factories\Factory;
    
    class UsersFactory extends Factory
    {
        /**
         * Define the model's default state.
         *
         * @return array<string, mixed>
         */
        public function definition(): array
        {
            return [
                'is_active' => $this->faker->boolean,
                'work_id' => str_pad($this->faker->unique()->numberBetween(1, 999999), 6, '0', STR_PAD_LEFT),
                'name' => $this->faker->name,
                // 可以根據需求添加其他欄位
            ];
        }
    }
    ```
    
3. 寫到測試裡，新增一函式來測試
    
    **tests\Feature\UserCreateApiTest.php**
    
    ```php
    <?php
    namespace Tests\Feature;
    use Tests\TestCase;
    use Database\Factories\UsersFactory;
    
    class UserCreateAPITest extends TestCase
    {
    ...
        /**
         * A basic feature test example.
         */
        public function test_create_user_api(): void
        {
            // Your test logic for creating a member
            $api_url = '/api/users';
            $api_method = "POST";
            //  執行 3 筆隨機資料測試
            $fake_data = UsersFactory::new()->count(3)->make();
    
            if (!empty($fake_data)) {
                foreach ($fake_data as $item) {
                    $response = $this
                        ->json($api_method, $api_url, $item->toArray());
                    $response->assertStatus(200)
                        ->assertJsonStructure([
                            'result',
                            'code',
                            'message',
                            'data',
                        ]);
                }
            }
        }
    }
    ```
    
4. 執行測試，會逐條列出裡面被測試的函式，綠燈則通過測試
    
    ```php
    $ php artisan test --filter UserCreateApiTest
    ```
    

## 測試流程的預處理 (以取得登入 token 為例)

1. 準備預先執行流程，將要取得的 token 在執行測試時到測試開始前先取得，然後存到共用參數中。
    
    **tests\TestCase.php**
    
    ```php
    <?php
    namespace Tests;
    
    use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
    abstract class TestCase extends BaseTestCase
    {
        use CreatesApplication;
        protected string $apiToken;
    
        protected function setUp():void
        {
            parent::setUp();
            // 取得 token
            $response = $this->get('api/login/', ['work_id'=>008899, 'pw'=>'132456798']);
            $this->apiToken = $response->json('data');
        }
    }
    ```
    
2. 加入到測試程式
