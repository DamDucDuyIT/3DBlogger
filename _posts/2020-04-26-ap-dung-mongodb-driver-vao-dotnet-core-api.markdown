---
layout: post
title: Áp dụng MongoDB Driver vào Dotnet Core API
date: 2019-04-26 12:00:30 +0300
image: 20.jpg
tags: [csharp, dotnet, back-end, mongodb]
---

Hiện tại, đa số lập trình viên Dotnet Core sử dụng Microsoft SQL làm cơ sở dữ liệu chính của dự án. Lý do có thể có rất nhiều: do quen thuộc với SQL query, tài liệu tìm hiểu nhiều, thiết lập và kết nối dễ dàng. Nhưng đặc biệt dotnet core có bộ thư viện Entity Framework Core hỗ trợ cực kỳ đắc lực trong quá trình truy xuất và kết hợp với LinQ để xử lý dữ liệu một cách dễ dàng.

Tuy nhiên, có thể nhiều lập trình viên trong quá trình thực hiện dự án cho khách hàng sẽ được yêu cầu sử dụng những NoSQL Database Engine mà ở đây chính là MongoDB. Với một số người, đây có thể là một việc vô cùng khó khăn. Cách thức viết query của SQL và NoSQL mặc dù có những điểm tương đồng, tuy nhiên tùy theo đặc trưng của mỗi loại sẽ có những điểm khác nhau và gây rắc rối cho người mới. Entity Framework Core thì lại không hỗ trợ cho MongoDB.

Từ đó, đối với những lập trình viên back-end Dotnet rất cần một bộ thư viện hỗ trợ họ lập trình, tuy xuất dữ liệu bằng ngôn ngữ C# những vẩn có thể giao tiếp và thực thi lệnh trên MongoDB. Đó chính là lý do mà chúng ta cần sử dụng đến MongoDB Driver. Bạn có thể lên [trang chủ][mongodb-driver-docs] của MongoDB Driver để tìm hiểu thêm chi tiết. Ngoài C#, nó còn hỗ trợ rất nhiều ngôn ngữ lập trình khác.

Vậy, trong bài viết này, tôi sẽ hướng dẫn các bạn cách thức để kết nối đến MongoDB và tạo một số API đơn giản để các bạn hiểu rõ cách thức hoạt động và liên kết giữa dotnet và MongoDB thông qua thư viện MongoDB Driver.

Việc đầu tiên chúng ta phải cài đặt MongoDB vào máy đã. CÁc bạn hãy truy cập vào đường dẫn này [https://www.mongodb.com/download-center/community][mongodb-download] và tải về bản cài đặt. Việc cài đặt khá là dễ dàng, chủ yếu Next, Next và Next. À khoan, trong quá trình cài đặt, nó sẽ có một cái checkbox hỏi chúng ta có cài đặt thêm MongoDB Compass không? Đây là một phần mềm hỗ trợ xem, chỉnh sửa và truy xuất dữ liệu trong MongoDB tương tự như Microsoft SQL Management. Tuy nhiên, theo ý kiến cá nhân của tôi, chúng ta nên sử dụng Robo 3T vì nó có giao diện thân thiện và dễ dàng sử dụng hơn. Bạn có thể tải về phần mềm này trong đường dẫn này [https://robomongo.org/download][robo3t-download]. Lưu ý download bản Robo3T nhé, vì nó miễn phí, còn bản Studio3T có tính phí đó.

Nếu như trong lúc cài đặt MongoDB mà bạn không thay đổi những thiết lập mặc định thì MongoDB sẽ được cài đặt trên port 27017. Bạn hãy khởi động Robo3T, trên popup hiển thị hãy chọn Create để tạo một kết nối đến MongoDB server bất kì. Mặc định phần mềm sẽ hiển thị server của MongoDB đang được cài đặt sẵn trên máy localhost với port 27017. Để tất cả mặc định sau đó nhấn Save. Sau đó thì kết nối với MongoDB server mà bạn vừa tạo thôi. Hiện tại thì server này chỉ có một vài database cấu hình thôi.

![]({{site.baseurl}}/img/17.jpg)

Để giải thích trước các từ ngữ chuyên về NoSQL Database Engine thì Collection chính là Table ở bên SQL Database Engine, Document chính là Row và Field chính là Column. Nắm được những định nghĩa này sẽ giúp bạn dễ hình dung hơn. Để hiểu rõ hơn về những tính chất của NoSQL và sự khác biệt với SQL, bạn có thể search trên Google, có nhiều nhiều bài viết bằng cả tiếng Việt và tiếng Anh giải thích vấn đề này. Tôi sẽ knêu ra ở đây mà có thể trong một bài viết khác vì nếu giải thích hết tất cả khía cạnh thì bài này sẽ khá dài và khá là lan man.

Bắt đầu tạo project nào. Ở đây tôi sẽ sử dụng Visual Studio 2019 và Dotnet Core 3.1 để thực hiện nha. Các bạn cứ làm bình thường với 'Create a new project' --> 'ASP.NET Core Web Application' --> Chọn tên và nơi lưu trữ --> Template sử dụng là API --> Create là xong

![]({{site.baseurl}}/img/18.jpg)

Bước tiếp theo là cài đặt thư viện MongoDB Driver. Các bạn có thể cài đặt thông qua commnand line bằng cách gõ

{% highlight tcl %}
dotnet add package MongoDB.Driver --version 2.10.3
{% endhighlight %}

Ngoài ra, nếu bạn thích sử dụng giao diện thì có thể cài đặt thông qua giao diện Nuget Packages của Visual Studio.

![]({{site.baseurl}}/img/19.jpg)

Bạn kiểm tra trong thư mục chứa source code sẽ có một file tên là appsettings.json. Đây sẽ là nơi lưu trữ đường dẫn cũng như thông tin của database mà chúng ta sẽ sử dụng. Bạn hãy cập nhật lại file này để trong bước tiếp theo có thể cài đặt kết nối đến database. Chúng ta khai báo những thông tin cơ bản như ConnectionString, tên Database, host và port. Vì chúng ta sử dụng chính MongoDB đã được cài đặt trong máy thông qua port 27017 ở bước trên nên cơ bản dữ liệu trong appsettings.json sẽ như sau:

{% highlight json %}
{
"MongoConnection": {
	"ConnectionString": "mongodb://localhost:27017",
	"Database": "DotnetMongoDBDriver",
	"Host": "localhost:27017",
	"Port": 27017
},
"Logging": {
	"LogLevel": {
		"Default": "Information",
		"Microsoft": "Warning",
		"Microsoft.Hosting.Lifetime": "Information"
	}
},
"AllowedHosts": "\*"
}
{% endhighlight %}

Tiếp theo, để code dễ sắp xếp và truy xuất dữ liệu dễ hơn. Chúng ta nên tạo 1 class DbContext để lưu trữ những dữ liệu trong appsetting.json. Việc map data sẽ được thực hiện khi project được khởi động. Vì thế, trong suốt quá trình hoạt động, chúng ta chỉ cần truy cập lại dữ liệu trong class này thôi

{% highlight csharp %}
namespace DotnetMongoDBDriver
{
    public class DbContext
    {
        public string ConnectionString { get; set; }
        public string Database { get; set; }
        public string Host { get; set; }
        public int Port { get; set; }
    }
}
{% endhighlight %}

Cài đặt nạp dữ liệu vào DbContext trong file Startup.cs trong phần ConfigureServices. Với việc cài đặt này thì hệ thống sẽ tự động dò tìm file appsettings.json để tìm những thông tin cần thiết và nạp dữ liêu tương ứng vào DbContext với mục đích phục vụ cho việc kết nối đến database trong tương lai

{% highlight csharp %}
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.Configure<DbContext>(options =>
        {     
           options.ConnectionString
                = Configuration.GetSection("MongoConnection:ConnectionString").Value;
           options.Database
                = Configuration.GetSection("MongoConnection:Database").Value;
           options.Host
                = Configuration.GetSection("MongoConnection:Host").Value;
           options.Port
                = Int32.Parse(Configuration.GetSection("MongoConnection:Port").Value);
        });
    }
{% endhighlight %}

Sau khi chuẩn bị mọi thứ, tương tự như các bạn sử dụng Entity Framework, chúng ta cần phải có 1 class ApplicationDbContext để lưu trữ thông tin và các bảng, giao thức kết nối hoặc phương thức truy xuất dữ liệu. Tùy theo phong cách của từng lập trình viên, nhưng tôi sẽ tạo 1 folder là data và lưu trữ những class cấu hình liên quan đến việc truy xuất, lưu trữ. Vì thế cả DbContext và ApplicationDbContext đều được tôi lưu trữ trong folder này

{% highlight csharp %}
    namespace DotnetMongoDBDriver.Data
    {
        public class ApplicationDbContext
        {
            private readonly IMongoDatabase _database = null;
            private IMongoClient _client = null;

            public ApplicationDbContext(IOptions<DbContext> configuration)
            {
                var connectionString = configuration.Value.ConnectionString;

                _client = new MongoClient(connectionString);

                if (_client != null)
                {
                    _database = _client.GetDatabase(configuration.Value.Database);
                }
            }
        }
    }
{% endhighlight %}

Giải thích một chút về source code ở trên. Chúng ta sẽ nhúng configuration từ DbContext mà đã được nạp dữ liệu từ bên Startup. Sử dụng new MongoClient để bắt đầu khởi tạo một kết nối đến với MongoDB. Nếu kết nối này được khởi tạo thành công và không xảy ra bất kỳ lỗi nào trong quá trình kết nối (is not null) thì sẽ nhảy đến bước tiếp theo là khởi tạo database. Các bạn yên tâm, đối với hàm 'GetDatabase', thư viện MongoDB.Driver sẽ kiểm tra là database đã tồn tại hay chưa, nếu chưa nó sẽ tạo database với tên nằm trong hai dấu ngoặc cho chúng ta một cách tự động.

Trong bài post này, để ví dụ cách áp dụng MongoDB một cách đơn giản, tôi cũng sẽ đưa ra một mô hình hệ thống cơ sở dữ liệu cũng đơn giản nhằm mục đích cho mọi người dễ hình dung. Ví dụ đó là: một ngành học trong trường Đại Học. Cụ thể ở đây là ngành công nghệ thông tin. Trong ngành học này ngoài thông tin chi tiết thì còn phải chứa những dữ liệu của rất nhiều sinh viên, giảng viên và môn học. Nói đến đây, nếu áp dụng SQL thì sơ sơ đã cần 4 table cho Ngành học, môn học, giảng viên và sinh viên, cùng với một số quan hệ một nhiều rối rắm (hoặc có thể không rối lắm nới những bạn đã có nhiều năm về xây dựng database :D). Tuy nhiên, với MongoDB, chúng ta chỉ cần có một Collection Ngành Học thôi, và Collection này sẽ chứa tất cả nhưng thông tin liên quan đến ngành học đó cũng như thông tin của những yếu tố phụ thuộc vào ngàn học này. Mà tính là một Collection vậy thôi, chứ lời khuyên chân thành của mình thì các bạn vẫn nên tạo ra 4 calss tượng trưng cho 4 đối tượng đi. Dễ quản lý, truy cập và sửa chữa sau này.

{% highlight csharp %}
namespace DotnetMongoDBDriver.Models
{
    public class Student
    {
        public string FullName { get; set; }
        public string StudentCode { get; set; }
        public string Address { get; set; }
        public DateTime BoD { get; set; }
        public string Phone { get; set; }
    }
}
{% endhighlight %}

{% highlight csharp %}
namespace DotnetMongoDBDriver.Models
{
    public class Lecturer
    {
        public string FullName { get; set; }
        public string StudentCode { get; set; }
        public string Address { get; set; }
        public DateTime BoD { get; set; }
        public string Phone { get; set; }
    }
}
{% endhighlight %}

{% highlight csharp %}
namespace DotnetMongoDBDriver.Models
{
    public class Subject
    {
        public string Name { get; set; }
        public string Description { get; set; }
        public int Year { get; set; }
        public string Location { get; set; }
    }
}
{% endhighlight %}

Khác với SQL, thì Id của từng Collection trong MongoDB sẽ là ObjectId nhé. Giải thích về ObjectID thì cũng hơi rắc rối (thật ra là mình không biết giải thích). Nói chung nó là một chuỗi string gồm kí tự và số. Giống như ID của SQL, ObjectId cũng sẽ được tạo tự động bằng MongoDB engine mỗi khi thêm mới một Collection.
{% highlight csharp %}
namespace DotnetMongoDBDriver.Models
{
    public class Major
    {
        public ObjectId Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public ICollection<Student> Students { get; set; }
        public ICollection<Lecturer> Lecturers { get; set; }
        public ICollection<Subject> Subjects { get; set; }
    }
}
{% endhighlight %}

Vì chúng ta chỉ cần đúng một Collection là Major (ngành học) chứa danh sách của giảng viên, sinh viên và môn học, nên chỉ cần khai báo duy nhất Collection này bên trong của file ApplicationDBContext là sẽ OK
{% highlight csharp %}
namespace DotnetMongoDBDriver.Data
{
    public class ApplicationDbContext
    {
        private readonly IMongoDatabase _database = null;
        private IMongoClient _client = null;
        public IMongoCollection<Major> Majors => _database.GetCollection<Major>("Major");
        public ApplicationDbContext(IOptions<DbContext> configuration)
        {
            var connectionString = configuration.Value.ConnectionString;

            _client = new MongoClient(connectionString);

            if (_client != null)
            {
                _database = _client.GetDatabase(configuration.Value.Database);

                _database.GetCollection<Major>("Major");
            }
        }
    }
}
{% endhighlight %}

Tương tự như GetDatabase thì GetCollection cũng sẽ kiểm tra và tự động tạo Collection Major ra. Ngoài ra, GetCollection còn có tác dụng như một hàm quey dữ liệu trong Collection, do đó chúng ta sẽ khai báo 1 biến public là Majors để sau này hỗ trợ chúng ta truy từ API.

Nói chung, những bước chuẩn về cấu hình và kết nối database cơ bản là đã hoàn thành. Lưu ý, đây mới chỉ là những hình thức cấu hình cơ bản nhất. Những dự án phức tạp hơn sẽ có nhiều cấu hình chuyên sâu hơn nhiều, ví dụ như tài khoản đăng nhập database, cấu hình bảo mật. mô hình cơ sở dữ liệu phức tạp hơn, liên kết nhiều database cùng lúc v.v... Nhưng thôi, với một bài chia sẻ kinh nghiệm và giúp làm quen với MongoDB thì nhiêu đây là vừa đủ, nếu cần thì mọi người có thể tìm kiếm thêm trên internet nhé, hoặc một bữa nào đó rảnh, mình sẽ làm 1 series chuyên sâu hơn về những vấn đề này.

OK. Đến bước bắt đầu tạo API rồi. Ai đang sữ dụng Visual Stuido thì đơn giản rồi. Chỉ cần chuột phải vào thư mục Controllers --> Add --> Controller --> API Controller và sau đó là đặt tên thôi.
{% highlight csharp %}
namespace DotnetMongoDBDriver.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class MajorController : ControllerBase
    {
        private ApplicationDbContext _context;

        public MajorController(IOptions<DbContext> configuration)
        {
            _context = new ApplicationDbContext(configuration);
        }
    }
}
{% endhighlight %}

Bước đến từng API nào. Cái đầu tiên sẽ là API lấy danh sách tất  cả ngành học trong database. Với sự hỗ trợ của MongoDB Driver thì query hay truy xuất dữ liệu của MongoDB cũng sẽ na ná như SQL vậy. À, nhớ thêm .AsQueryable() nhé, phải có thì nó mới gợi ý những hàm hoặc tính năng giống với Entity Framework
{% highlight csharp %}
       [HttpGet]
        public async Task<IActionResult> Get()
        {
            var majors = await _context.Majors
                        .AsQueryable()
                        .ToListAsync();

            return Ok(majors);
        }
{% endhighlight %}

Tiếp theo sẽ là API lấy theo Id. Vì id của document trong Collection của MongoDB là kiểu OnjectId. Mà chúng ta không thể cung cấp ObjectId trong URL của API được. Do đó, kiểu bắt buộc phải là kiểu string. Sau đó dùng new ObjectId để chuyển nó thành ObjectId. Ngoài ra, với việc tìm kiếm đối tượng theo điều kiện. MongoDB Driver cung cấp cho chúng ta hàm .Find khá là hay (nó tương tự như .Where vậy đó) 
{% highlight csharp %}
        [HttpGet("{id}", Name = "Get")]
        public async Task<IActionResult> Get(string id)
        {
            var major = await _context.Majors
                   .Find(m => m.Id == new ObjectId(id))
                   .FirstOrDefaultAsync();

            return Ok(major);
        }
{% endhighlight %}

Đối với API POST để thêm mới đối tượng thì MongoDB Driver cung cấp cho chúng ta .InsertOneAsync rất là dễ sữ dụng. Và đặc biệt, sau khi hàm này chạy thành công, bạn không cần phải SaveChange hay SaveChangeAsync giống của Entity Framework. Điều này đôi có mặt tốt và cũng có mặt xấu của nó.
{% highlight csharp %}
     [HttpPost]
        public async Task<IActionResult> Post([FromBody] Major major)
        {
            await _context.Majors.InsertOneAsync(major);

            return Ok(major);
        }
{% endhighlight %}

Với API cập nhật đối tượng - PUT, tương tự như POST, .ReplaceOneAsync đượ cung cấp để thực hiện tác vụ này.
{% highlight csharp %}
        [HttpPut("{id}")]
        public async Task<IActionResult> Put(string id, [FromBody] Major major)
        {
            await _context.Majors.ReplaceOneAsync(n => n.Id == new ObjectId(id), major);

            return Ok(major);
        }
{% endhighlight %}

API cuối cùng là DELETE, trong những dự án chạy thật, chúng ta không nên xóa hẳn đối tượng mà chỉ nên chuyển nó đi đâu đó hoặc có properties xác nhận đã xóa hay chưa thôi. Tuy nhiên mình cũng giới thiệu cách xóa hẳn đối tượng trong MongoDB.
{% highlight csharp %}
   [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(string id)
        {
            await _context.Majors.DeleteOneAsync(n => n.Id == new ObjectId(id));

            return Ok();
        }
{% endhighlight %}

Vậy là chúng ta đã xong những API căn bản. Ngoài những API này, MongoDB Driver còn cung cấp rất nhiều tính năng mạnh mẽ và hữ dụng khác như cập nhật đồng thời nhiều document cùng lúc, filter theo data nằm trong array của docuemnt v.v.. Tuy nhiên, việc đầu tiên là nắm vững căn bản đã, sau đó mới nghiên cứu, mở rộng và phát triển trong tương lai.

Mong rằng bài blog này có thể giúp cmọi người có thêm được một vài kiến thức và có thể áp dụng MongoDB Driver cũng như sử dụng MongoDB vào những dự án trong tương lai. Link github chứa source tham khảo: [Github][github]
[mongodb-driver-docs]: https://docs.mongodb.com/drivers/
[mongodb-download]: https://www.mongodb.com/download-center/community/
[robo3t-download]: https://robomongo.org/download/
[github]: https://github.com/DamDucDuyIT/DotnetMongoDBDriver/
