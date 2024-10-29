---
title: File manipulation and conversion
summary: Learn how to manipulate and convert file records in code
icon: file-medical-alt
---

# File manipulation and conversion

Asset storage is provided out of the box via a [Flysystem](https://flysystem.thephpleague.com/docs/) backend store. This abstraction allows for files to be stored in any number of different ways, such as storing them in the cloud, so you cannot rely on having a local file path in order to get and manipulate the contents of any given asset.

Silverstripe CMS provides a well-abstracted API for creating, manipulating, and storing assets.

See [images](./images/) for some image-specific manipulation methods.

## Creating new files in PHP

When working with files in PHP you can upload a file into a [`File`](api:SilverStripe\Assets\File) dataobject
using one of the below methods:

| Method                     | Description                             |
| -------------------------- | --------------------------------------- |
| `File::setFromLocalFile`   | Load a local file into the asset store  |
| `File::setFromStream`      | Will store content from a stream        |
| `File::setFromString`      | Will store content from a binary string |

For example:

```php
use SilverStripe\Assets\File;

// Store a file named "example-file.txt".
$fileRecord = File::create();
$fileRecord->setFromString('This is some file content', 'example-file.txt');
$fileRecord->write();
```

If you want to store your file in a [`DBFile`] field directly, or you don't want to use the `File` model for some reason,
you can also use the default [`AssetStore`](api:SilverStripe\Assets\Storage\AssetStore) directly:

```php
use SilverStripe\Assets\File;
use SilverStripe\Assets\Storage\AssetStore;
use SilverStripe\Core\Injector\Injector;
use SilverStripe\ORM\FieldType\DBField;

// Store a file named "example-file.txt".
$store = Injector::inst()->get(AssetStore::class);
$result = $store->setFromString('This is some file content', 'example-file.txt');

// Save a database record that points to the stored file.
// Note that we pull the file name from the result because the asset store might have renamed it.
$dbFile = DBField::create_field('DBFile', $result);
$fileRecord = File::create(['Name' => $result['Filename'], 'File' => $dbFile]);
$fileRecord->write();
```

### Storage conflict resolution

When storing new files, it's possible to determine the mechanism the backend should use when it encounters
an existing file name pattern. The conflict resolution to use can be passed into the third parameter of the
above methods (after content and filename). The available constants are:

| Constant                            | If an existing file is found then:  |
| ----------------------------------- | ----------------------------------- |
| `AssetStore::CONFLICT_EXCEPTION`    | An exception will be thrown         |
| `AssetStore::CONFLICT_OVERWRITE`    | The existing file will be replaced  |
| `AssetStore::CONFLICT_RENAME`       | The backend will choose a new name  |
| `AssetStore::CONFLICT_USE_EXISTING` | The existing file will be used      |

If no conflict resolution scheme is chosen, or an unsupported one is requested, then the backend will choose one.
The default asset store supports each of these.

The conflict resolution is passed in to the `config` argument like so:

```php
use SilverStripe\Assets\File;
use SilverStripe\Assets\Storage\AssetStore;

// Store a file named "example-file.txt".
$fileRecord = File::create();
$fileRecord->setFromString(
    'This is some file content',
    'example-file.txt',
    // If a file with that name already exists, let the file store rename this one.
    config: ['conflict' => AssetStore::CONFLICT_RENAME]
);
$fileRecord->write();
```

See [file storage](./file_storage/) for more details about the way files are stored.

## Accessing files via PHP

As with storage, there are also different ways of loading the content (or properties) of the file:

| Method                   | Description                                                |
| ------------------------ | ---------------------------------------------------------- |
| `File::getStream`        | Will get an output stream of the file content              |
| `File::getString`        | Gets the binary content                                    |
| `File::getURL`           | Gets the URL for this resource. May or may not be absolute |
| `File::getAbsoluteURL`   | Gets the absolute URL to this resource                     |
| `File::getMimeType`      | Get the mime type of this file                             |
| `File::getMetaData`      | Gets other metadata from the file as an array              |
| `File::getFileType`      | Return the type of file for the given extension            |

### Additional file types

Silverstripe CMS has a pre-defined list of common file types. `File::getFileType` will return "unknown" for files outside that list.

You can add your own file extensions and their description with the following configuration:

```yml
SilverStripe\Assets\File:
  file_types:
    ai: 'Adobe Illustrator'
    psd: 'Adobe Photoshop File'
```

## Renaming and moving files

In order to move or rename a file you can simply update the `Name` property, or assign the `ParentID` to a new
folder. Please note that these modifications are made simply on the draft stage, and will not be copied
to live until a publish is made via the CMS (either on this object, or cascading from a parent).

When files are renamed using the ORM, all file variants are automatically renamed at the same time.

```php
use SilverStripe\Assets\File;

$file = File::get()->filter('Name', 'oldname.jpg')->first();
if ($file) {
    // The below will move 'oldname.jpg' and 'oldname__variant.jpg'
    // to 'newname.jpg' and 'newname__variant.jpg' respectively
    $file->Name = 'newname.jpg';
    $file->write();
}
```

Note that you can cause the file to be moved immediately by [setting the Versioned reading mode](api:SilverStripe\Versioned\Mode\Versioned::set_reading_mode()) to draft temporarily.

```php
use SilverStripe\Assets\File;
use SilverStripe\Versioned\Mode\Versioned;

$file = File::get()->filter('Name', 'oldname.jpg')->first();
if ($file) {
    // The below will immediately move 'oldname.jpg' and 'oldname__variant.jpg'
    // to 'newname.jpg' and 'newname__variant.jpg' respectively
    $file->Name = 'newname.jpg';
    Versioned::withVersionedMode(function () use ($file) {
        Versioned::set_reading_mode('Stage.' . Versioned::DRAFT);
        $file->write();
        $file->publishSingle();
    });
}
```

## Convert a file to a different format {#file-conversion}

### The high-level API {#file-conversion-highlevel}

The [`File`](api:SilverStripe\Assets\File) and [`DBFile`](api:SilverStripe\Assets\Storege\DBFile) classes share a trait which implements a [`Convert()`](api:SilverStripe\Assets\ImageManipulation::Convert()) method.

When you call the method and pass in a file extension, if there is a converter registered which can handle conversion of your file to that format, the conversion will be performed.

> [!TIP]
> The extension in the URL of the resulting file is case sensitive - if you use all caps, the URL for the file will have a file extension in all caps.

You can call the method in PHP or in templates. If the format you are converting *to* is an image format supported by Intervention Image, you can chain any of the [image manipulation methods](./images/) afterward.

```php
// Convert an image to webp format and apply the FitMax manipulation to the result
$result = $this->MyImage()->Convert('webp')->FitMax(100, 100);
```

```ss
<%-- Convert an image to webp format and apply the FitMax manipulation to the result --%>
$MyImage.Convert('webp').FitMax(100, 100)
```

> [!WARNING]
> Don't include a `.` before the extension. For example, this will not work:
>
> ```php
> $result = $this->MyImage()->Convert('.webp');
> ```

If the file conversion fails, or there is no converter registered which supports it, it will return `null`, and nothing will be displayed in the template. The error will be logged.

#### How everything is wired together

There is a [`FileConverterManager`](api:SilverStripe\Assets\Conversion\FileConverterManager) class which has an array of classes that implement the [`FileConverter`](api:SilverStripe\Assets\Conversion\FileConverter) interface. These are stored in the [`FileConverterManager.converters`](api:SilverStripe\Assets\Conversion\FileConverterManager->converters) configuration array.

When you call the `Convert()` method on a `File` or `DBFile` object, it tells the `FileConverterManager` to perform the conversion. The `FileConverterManager` loops through the registered converters and calls the [`FileConverter::supportsConversion()`](api:SilverStripe\Assets\Conversion\FileConverter::supportsConversion()) method on each of them until it finds one that can support the requested conversion.

If it finds a converter that can support the conversion, it calls the [`FileConverter::Convert()`](api:SilverStripe\Assets\Conversion\FileConverter::Convert()) method on that converter and returns the result.

If no converter can be found (or if the conversion fails), a [`FileConverterException`](api:SilverStripe\Assets\Conversion\FileConverterException) is thrown.

### The low-level API {#file-conversion-lowlevel}

You can use the [`manipulateExtension()`](api:SilverStripe\Assets\ImageManipulation::manipulateExtension()) method on any `File` or `DBFile` object to create a variant with a different file extension than the original.

This can be very useful if you want to convert a file to a different format for the user to download or view, while leaving the original file intact. Some examples of when you might want this are:

- Generating thumbnails for videos, documents, etc
- Converting images to `.webp` for faster page load times
- Converting documents to `.pdf` so downloaded documents are more portable

#### Making our own `FileConverter`

Converting between image formats is the easiest example, because we can let [Intervention Image](https://image.intervention.io/v3) do the heavy lifting for us. Note that there is a built in [`InterventionImageFileConverter`](api:SilverStripe\Assets\Conversion\InterventionImageFileConverter) class which does this already, but we'll use this as an example for how to create our own `FileConverter`.

The `FileConverter` interface requires us to implement two methods:

- `supportsConversion()` must return a boolean value indicating whether it would support a given conversion or not.
- `convert()` performs the actual conversion, or throws a [`FileConverterException`](api:SilverStripe\Assets\Conversion\FileConverterException) on failure.

```php
namespace App\Conversion;

use Intervention\Image\Exceptions\RuntimeException;
use SilverStripe\Assets\Conversion\FileConverter;
use SilverStripe\Assets\Conversion\FileConverterException;
use SilverStripe\Assets\File;
use SilverStripe\Assets\Storage\AssetStore;
use SilverStripe\Assets\Storage\DBFile;

class ImageFileConverter implements FileConverter
{
    public function supportsConversion(string $fromExtension, string $toExtension, array $options = []): bool
    {
        $supported = true;
        /* some logic here to check if this conversion is supported */
        return $supported;
    }

    public function convert(DBFile|File $from, string $toExtension, array $options = []): DBFile
    {
        $from = $this->getOwner();
        try {
            return $from->manipulateExtension(
                $toExtension,
                function (AssetStore $store, string $filename, string $hash, string $variant) use ($from) {
                    $backend = $from->getImageBackend();
                    $config = ['conflict' => AssetStore::CONFLICT_USE_EXISTING];
                    $tuple = $backend->writeToStore($store, $filename, $hash, $variant, $config);
                    return [$tuple, $backend];
                }
            );
        } catch (RuntimeException $e) {
            throw new FileConverterException('Failed to convert: ' . $e->getMessage(), $e->getCode(), $e);
        }
    }
}
```

And we need to register the converter in the [`FileConverterManager`](api:SilverStripe\Assets\Conversion\FileConverterManager).

The `Before: '#assetsconversion'` part here is optional - it can be used to mark your converter as a higher priority than the one defined in the `assetsconversion` configuration in `silverstripe/assets`.

```yml
---
Before: '#assetsconversion'
---
SilverStripe\Assets\Conversion\FileConverterManager:
  converters:
    - 'App\Conversion\MyImageFileConverter'
```

Let's look at what's actually happening in the `convert()` method, piece by piece.

```php
return $from->manipulateExtension($toExtension /* ... */);
```

We call the `manipulateExtension()` method and pass in the file extension we want to convert our image to. If that variant file already exists, it won't call the callback method - the asset store system won't generate the file again if it already exists.

We'll be returning the result of this manipulation, which will be a `DBFile` containing all of the relevant information about our new variant.

```php
function (AssetStore $store, string $filename, string $hash, string $variant) use ($from) {
    $backend = $from->getImageBackend();
    // ...
};
```

We define a callback, which will be called by `manipulateExtension()` if our variant file doesn't exist yet. This callback will be responsible for generating and storing the variant file.

The parameters for the callback function are as follows:

| Type | Name | Description |
| -----| ---- | ----------- |
| [`AssetStore`](api:api:SilverStripe\Assets\Storage\AssetStore) | store | The mechanism used to store the actual file |
| `string` | filename | The name of the original file, including the original file extension |
| `string` | hash | An sha1 hash of the original file content |
| `string` | variant | A base64 encoded string with information about the variant file you're creating |

We also want access to the original file record here so that we can use its [`Image_Backend`](api:SilverStripe\Assets\Image_Backend) to store the new file and do the conversion for us.

```php
$config = ['conflict' => AssetStore::CONFLICT_USE_EXISTING];
$tuple = $backend->writeToStore($store, $filename, $hash, $variant, $config);
```

As mentioned earlier, Intervention Image will be converting the image for us. The `$backend` variable (unless you've replaced it with something else) is an instance of [`InterventionBackend`](api:SilverStripe\Assets\InterventionBackend) which implements `Image_Backend` and uses the Intervention Image API.

The `$variant` variable holds information about the file conversion we want to make, so this line is just us saying "take this image, convert it to this new file type, and store the result."

Notice that we're using the `CONFLICT_USE_EXISTING` [conflict resolution strategy](#storage-conflict-resolution). Our callback *shouldn't* be called if our variant file already exists, but just in case it does, we can just use the existing file instead of generating a new one.

The value returned from `writeToStore()` is an associative array with information about the new variant file you've created.

```php
return [$tuple, $backend];
```

Our callback returns both the information about the variant file and the `Image_Backend` object we used to generate it. Returning the `Image_Backend` here is important, because it will be used to perform any image-related manipulations we want to perform afterwards.

```php
try {
    // ...
} catch (RuntimeException $e) {
    throw new FileConverterException('Failed to convert: ' . $e->getMessage(), $e->getCode(), $e);
}
```

Finally, if Intervention Image failed to perform the conversion for any reason, we catch its exception and wrap it in the expected `FileConverterException`.

As described in [the high-level API](#file-conversion-highlevel) above, you can use the `Convert()` method in PHP code or in templates on any instance of `File` or `DBFile`, and this converter will be used to perform the conversion.

For example, if your page has a relation called `MyImage` to an `Image` record:

```ss
$MyImage.Convert('webp').ScaleWidth(150)
```

See [images](./images/) for more information about image-specific manipulation methods.

#### Converting between other formats

Converting between other formats (including a non-image to an image) is a little bit more involved, because we have to find a third-party library that will do the conversion for us and then store the new content.

Below are two examples for these conversions - one where a file is converted to an image, and another where a file is converted to a PDF.

These examples won't include performing the actual conversion from one format to another, because that would need to be handled by some third-party library. Instead, they demonstrate how to use the `manipulateExtension()` API to store the converted files as variants.

```php
namespace App\Conversion;

use SilverStripe\Assets\Conversion\FileConverter;
use SilverStripe\Assets\Conversion\FileConverterException;
use SilverStripe\Assets\File;
use SilverStripe\Assets\Image_Backend;
use SilverStripe\Assets\Storage\AssetStore;
use SilverStripe\Assets\Storage\DBFile;
use SilverStripe\Core\Injector\Injector;

class MyFileConverter implements FileConverter
{
    public function supportsConversion(string $fromExtension, string $toExtension, array $options = []): bool
    {
        $supported = true;
        /* some validation here to check if this conversion is supported */
        return $supported;
    }

    public function convert(DBFile|File $from, string $toExtension, array $options = []): DBFile
    {
        $fromExtension = $from->getExtension();
        if (!$this->supportsConversion($fromExtension, $toExtension, $options)) {
            throw new FileConverterException(
                "Conversion from '$fromExtension' to '$toExtension with those options is not supported."
            );
        }

        // Handle conversion to PDF
        if (strtolower($toExtension) === 'pdf') {
            return $from->manipulateExtension(
                $toExtension,
                function (AssetStore $store, string $filename, string $hash, string $variant) {
                    $tmpFilePath = /* some conversion logic goes here */;
                    $config = ['conflict' => AssetStore::CONFLICT_USE_EXISTING];
                    $tuple = $store->setFromLocalFile($tmpFilePath, $filename, $hash, $variant, $config);
                    return [$tuple, null];
                }
            );
        }

        // Handle conversion to image
        return $from->manipulateExtension(
            $toExtension,
            function (AssetStore $store, string $filename, string $hash, string $variant) {
                $tmpFilePath = /* some conversion logic goes here */;
                $backend = Injector::inst()->create(Image_Backend::class);
                $backend->loadFrom($tmpFilePath);
                $config = ['conflict' => AssetStore::CONFLICT_USE_EXISTING];
                $tuple = $backend->writeToStore($store, $filename, $hash, $variant, $config);
                return [$tuple, $backend];
            }
        );
    }
}
```

After registering the converter with `FileConverterManager`, it will be available via the `Convert()` method on any file record.

```yml
SilverStripe\Assets\Conversion\FileConverterManager:
  converters:
    - 'App\Conversion\MyFileConverter'
```

Okay, now lets step through those conversions and take a look at what's going on. We'll only look at the parts that are different from the image-to-image conversion [we looked at earlier](#making-our-own-fileconverter).

##### Converting something to an image

The main difference between converting from one image to another compared with converting a non-image to an image, is that you have to get a third-party to perform the conversion for you.

```php
$tmpFilePath = /* some conversion logic goes here */;
$backend = Injector::inst()->create(Image_Backend::class);
$backend->loadFrom($tmpFilePath);
```

After the actual file conversion has happened, and you have the new file contents stored in some temporary location (e.g. using [`tmpfile`](https://www.php.net/manual/en/function.tmpfile.php)), we need to load that content into a `Image_Backend`. Unlike before, we don't have an existing image, so we need to get a new backend using the `Injector`.

The rest is the same as when we were converting from an image - we still get Intervention Image to store the variant file for us, and we make sure to include the `Image_Backend` object in our returned value.

##### Converting something to something else

When the format we're converting to is *not* an image, things are a little simpler. Again, we have to perform the conversion ourselves.

```php
$tmpFilePath = /* some conversion logic goes here */;
$config = ['conflict' => AssetStore::CONFLICT_USE_EXISTING];
$tuple = $store->setFromLocalFile($tmpFilePath, $filename, $hash, $variant, $config);
```

Then, since we're not saving an image, we can just use the [normal asset store logic](#creating-new-files-in-php).

```php
return [$tuple, null];
```

Our new file variant isn't an image in this case, so we won't need access to the image manipulation methods provided by an `Image_Backend`. So instead, we just put `null` in its place.
