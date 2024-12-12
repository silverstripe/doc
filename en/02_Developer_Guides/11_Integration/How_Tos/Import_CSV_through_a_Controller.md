---
title: Import CSV Data through a Controller
summary: Data importing through the frontend
icon: upload
---

# Import CSV data through a controller

You can have more customised logic and interface feedback through a custom controller. Let's create a simple upload
form (which is used for `MyDataObject` instances). You can access it through
`https://www.example.com/MyController/?flush=all`.

```php
namespace App\Control;

use App\Model\MyDataObject;
use SilverStripe\Control\Controller;
use SilverStripe\Dev\CsvBulkLoader;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\FileField;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;

class MyController extends Controller
{
    private static $url_segment = 'my_controller';

    private static $allowed_actions = [
        'getForm',
    ];

    private static $url_handlers = [
        'Form' => 'getForm',
    ];

    protected $template = 'BlankPage';

    public function getForm()
    {
        $form = Form::create(
            $this,
            'Form',
            FieldList::create(
                FileField::create('CsvFile', false)
            ),
            FieldList::create(
                FormAction::create('doUpload', 'Upload')
            )
        );
        return $form;
    }

    public function doUpload($data, $form)
    {
        $loader = CsvBulkLoader::create(MyDataObject::class);
        $results = $loader->load($_FILES['CsvFile']['tmp_name']);
        $messages = [];

        if ($results->CreatedCount()) {
            $messages[] = sprintf('Imported %d items', $results->CreatedCount());
        }

        if ($results->UpdatedCount()) {
            $messages[] = sprintf('Updated %d items', $results->UpdatedCount());
        }

        if ($results->DeletedCount()) {
            $messages[] = sprintf('Deleted %d items', $results->DeletedCount());
        }

        if (!$messages) {
            $messages[] = 'No changes';
        }

        $form->sessionMessage(implode(', ', $messages), 'good');

        return $this->redirectBack();
    }
}
```

> [!CAUTION]
> If the file you're loading data from is uploaded by a user, you should pass `true` to the [`CsvBulkLoader::setCheckPermissions()`](api:SilverStripe\Dev\CsvBulkLoader::setCheckPermissions()) method. Otherwise, permissions will not be respected and the user may alter data in ways they would otherwise not be permitted to.
