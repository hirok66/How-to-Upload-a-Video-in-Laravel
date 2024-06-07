# How-to-Upload-a-Video-in-Laravel
How to Upload a Video in Laravel


Step 1: Create a new Laravel project using composer. You can give it any name but in this case, we will name it videoApp.

composer create-project --prefer-dist laravel/laravel videoApp

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 2: In the videoApp project create a migration to create a videos table.

cd videoApp
php artisan make:migration create_videos_table --create=videos


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 3: In the migration add the fields video and title.

<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateVideosTable extends Migration
{
  public function up()
  {
    Schema::create('videos', function (Blueprint $table) {
      $table->bigIncrements('id');
      $table->string('title');
      $table->string('video');
      $table->timestamps();
  });
  }
  public function down()
  {
    Schema::dropIfExists('videos');
  }
}

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

Step 4: Next, create a model called Video.

php artisan make:model Video

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 5: In the model add the video fields and title fields in the fillable array.

<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Video extends Model
{
  protected $table = 'videos';
  protected $fillable = [
      'title', 'video'
  ];
}
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 6: In the config/ folder, add a new disk called my_files in a file called filesystems.php.

'disks' => [
   'local' => [
      'driver' => 'local',
      'root' => storage_path('app'),
   ],
   'public' => [
      'driver' => 'local',
      'root' => storage_path('app/public'),
      'url' => env('APP_URL').'/storage',
      'visibility' => 'public',
   ],
   's3' => [
      'driver' => 's3',
      'key' => env('AWS_ACCESS_KEY_ID'),
      'secret' => env('AWS_SECRET_ACCESS_KEY'),
      'region' => env('AWS_DEFAULT_REGION'),
      'bucket' => env('AWS_BUCKET'),
      'url' => env('AWS_URL'),
   ],
   'my_files' => [
      'driver' => 'local',
      'root'   => public_path() . '/'
   ]
],
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 5: Next, create a controller and name it VideoController.

php artisan make:controller VideoController
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 6: To the VideoController, add a function called uploadVideo.

<?php
namespace App\Http\Controllers\Admin;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;
use App\Video;
use Storage;
class VideoController extends Controller
{
   public function uploadVideo(Request $request)
   {
      $this->validate($request, [
         'title' => 'required|string|max:255',
         'video' => 'required|file|mimetypes:video/mp4',
   ]);
   $video = new Video;
   $video->title = $request->title;
   if ($request->hasFile('video'))
   {
     $path = $request->file('video')->store('videos', ['disk' =>      'my_files']);
    $video->video = $path;
   }
   $video->save();
   
  }
}

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 7: Finally in the web.php file in the routes folder

<?phpuse Illuminate\Http\Request;
Route::post('uploadVideo', 'VideoController@uploadVideo')->name('videos.uploadedVideo');

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Step 8: Create a file called uploadVideo.blade.php.

<!DOCTYPE html>
<html>
<head>
 <meta charset="utf-8">
 <meta name="viewport" content="width=device-width, initial-scale=1,   shrink-to-fit=no">
<title>Video upload</title>
</head>
<body>
<div>
<h3>Upload a video</h3>
<hr>
<form method="POST" action="{{ route('videos.uploadVideo') }}" enctype="multipart/form-data" >
{{ csrf_field() }}
<div >
<label>Title</label>
<input type="text" name="title" placeholder="Enter Title">
</div>
<div >
<label>Choose Video</label>
<input type="file"  name="video">
</div>
<hr>
<button type="submit" >Submit</button>
</form>
</div>
</body>
</html>
