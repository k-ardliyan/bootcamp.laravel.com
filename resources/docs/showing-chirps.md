# Showing Chirps

In the previous step we added the ability to create Chirps, now we're ready to display them!

## Controller

Let's update the `index()` method our `ChirpController` to pass Chirps from every user to our Index page.

```php filename=app/Http/Controllers/ChirpController.php
<?php
// [tl! collapse:start]
namespace App\Http\Controllers;

use App\Models\Chirp;
use Illuminate\Http\Request;
use Inertia\Inertia;
// [tl! collapse:end]
class ChirpController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index(Request $request)
    {
        return Inertia::render('Chirps/Index', [
            //
            'chirps' => Chirp::with('user:id,name')->latest()->get(),// [tl! remove:-1,1 add]
        ]);
    }
    // [tl! collapse:start]
    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $request->user()->chirps()->create([
            'message' => $request->input('message'),
        ]);

        return redirect(route('chirps.index'));
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function show(Chirp $chirp)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function edit(Chirp $chirp)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Chirp $chirp)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function destroy(Chirp $chirp)
    {
        //
    }
    // [tl! collapse:end]
}
```

## Relationship

We've instructed Laravel to return the `id` and `name` property from the `user` relationship so that we can display the name of the Chirp author, without returning other potentially sensitive information such as the users email address. The `user` relationship hasn't been defined yet, so let's add a new "belongs to" relationship on our `Chirp` model:

```php filename=app/Models/Chirp.php
<?php
// [tl! collapse:start]
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
// [tl! collapse:end]
class Chirp extends Model
{
    // [tl! collapse:start]
    use HasFactory;

    protected $fillable = [
        'message',
    ];
    // [tl! collapse:end]
    public function user()// [tl! add:start]
    {
        return $this->belongsTo(User::class);
    }// [tl! add:end]
}
```

This is the inverse relationship for the "has many" relationship we created earlier on the `User` model.

## Updating our component

We'll also create a `Chirp` component for our front-end:

```vue tab=Vue filename=resources/js/Components/Chirp.vue
<script setup>
defineProps(['chirp']);
</script>

<template>
    <div class="p-6 flex space-x-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-600 -scale-x-100" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
        </svg>
        <div class="flex-1">
            <div class="text-gray-800">{{ chirp.user.name }} <small class="text-sm text-gray-600">{{ chirp.created_at }}</small></div>
            <p class="mt-4 text-lg text-gray-900">{{ chirp.message }}</p>
        </div>
    </div>
</template>
```

```javascript tab=React filename=resources/js/Components/Chirp.jsx
import React from 'react';

export default function Chirp({ chirp }) {
    return (
        <div className="p-6 flex space-x-2">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-gray-600 -scale-x-100" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth="2">
                <path strokeLinecap="round" strokeLinejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
            </svg>
            <div className="flex-1">
                <div className="text-gray-800">{chirp.user.name} <small class="text-sm text-gray-600">{chirp.created_at}</small></div>
                <p className="mt-4 text-lg text-gray-900">{chirp.message}</p>
            </div>
        </div>
    );
}
```

And finally, we will update our `Chirps/Index` page component to accept the `chirps` prop and render them below our form using our new component:

```vue tab=Vue filename=resources/js/Pages/Chirps/Index.vue
<script setup>
import BreezeAuthenticatedLayout from '@/Layouts/Authenticated.vue';
import BreezeButton from '@/Components/Button.vue';
import BreezeInputError from '@/Components/InputError.vue';
import Chirp from '@/Components/Chirp.vue';// [tl! add]
import { Head } from '@inertiajs/inertia-vue3';
import { useForm } from '@inertiajs/inertia-vue3';

defineProps(['chirps']);// [tl! add]

const form = useForm({
    message: '',
});
</script>

<template>
    <Head title="Dashboard" />

    <BreezeAuthenticatedLayout>
        <div class="max-w-2xl mx-auto p-4 sm:p-6 lg:p-8">
            <form @submit.prevent="form.post(route('chirps.store'), { onSuccess: () => form.reset() })">
                <textarea
                    v-model="form.message"
                    placeholder="What's on your mind?"
                    class="block w-full border-gray-300 focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 rounded-md shadow-sm"
                ></textarea>
                <BreezeInputError :message="form.errors.message" class="mt-2" />
                <BreezeButton class="mt-4">Chirp</BreezeButton>
            </form>

            <div class="mt-6 bg-white shadow-sm rounded-lg divide-y"><!-- [tl! add:start] -->
                <Chirp
                    v-for="chirp in chirps"
                    :key="chirp.id"
                    :chirp="chirp"
                />
            </div><!-- [tl! add:end] -->
        </div>
    </BreezeAuthenticatedLayout>
</template>
```

```javascript tab=React filename=resources/js/Pages/Chirps/Index.jsx
import React from 'react';
import Authenticated from '@/Layouts/Authenticated';
import Button from '@/Components/Button';
import Chirp from '@/Components/Chirp';// [tl! add]
import InputError from '@/Components/InputError';
import { useForm, Head } from '@inertiajs/inertia-react';

export default function Index(props) {
    const { data, setData, post, processing, reset, errors } = useForm({
      message: '',
    });

    const submit = (e) => {
        e.preventDefault();
        post(route('chirps.store'), { onSuccess: () => reset() })
    };

    return (
        <Authenticated auth={props.auth}>
            <Head title="Chirps" />

            <div className="max-w-2xl mx-auto p-4 sm:p-6 lg:p-8">
                <form onSubmit={submit}>
                    <textarea
                        value={data.message}
                        placeholder="What's on your mind?"
                        className="block w-full border-gray-300 focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 rounded-md shadow-sm"
                        onChange={e => setData('message', e.target.value)}
                    ></textarea>
                    <InputError message={errors.message} className="mt-2" />
                    <Button className="mt-4" disabled={processing}>Chirp</Button>
                </form>

                <div className="mt-6 bg-white shadow-sm rounded-lg divide-y">
                    {chirps.map(chirp =>
                        <Chirp key={chirp.id} chirp={chirp} />
                    )}
                </div>
            </div>
        </Authenticated>
    );
    // [tl! add:-8,5 .hidden]
}
```

Refresh the page at [http://localhost/chirps](http://localhost/chirps) to see the message you chirped earlier!

<img src="/img/screenshots/chirp-index.png" alt="Chirp listing" class="rounded-lg" />

### Date formatting

The dates we get back from Laravel are computer-friendly, but not very human-friendly. We can [customize the serialization format](https://laravel.com/docs/9.x/eloquent-serialization#date-serialization) that Laravel uses, but it's often useful to keep the date in a computer-friendly format for our front-end code. Let's take advantage of that format by using the popular [Day.js](https://day.js.org) library to display relative dates beside our Chirps!

First, install the `dayjs` NPM package:

```sh
npm install dayjs
```

Then we can use this in our `Chirp` component:

```vue tab=Vue filename=resources/js/Components/Chirp.vue
<script setup>
import dayjs from 'dayjs';// [tl! add]
import relativeTime from 'dayjs/plugin/relativeTime';// [tl! add]

dayjs.extend(relativeTime);// [tl! add]

defineProps(['chirp']);
</script>

<template>
    <div class="p-6 flex space-x-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-600 -scale-x-100" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
        </svg>
        <div class="flex-1">
            <div class="text-gray-800">{{ chirp.user.name }} <small class="text-sm text-gray-600">{{ chirp.created_at }}</small></div><!-- [tl! remove] -->
            <div class="text-gray-800">{{ chirp.user.name }} <small class="text-sm text-gray-600">{{ dayjs(chirp.created_at).fromNow() }}</small></div><!-- [tl! add] -->
            <p class="mt-4 text-lg text-gray-900">{{ chirp.message }}</p>
        </div>
    </div>
</template>
```

```javascript tab=React filename=resources/js/Components/Chirp.jsx
import React from 'react';
import dayjs from 'dayjs';// [tl! add]
import relativeTime from 'dayjs/plugin/relativeTime';// [tl! add]

dayjs.extend(relativeTime);// [tl! add]

export default function Chirp({ chirp }) {
    return (
        <div className="p-6 flex space-x-2">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-gray-600 -scale-x-100" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth="2">
                <path strokeLinecap="round" strokeLinejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
            </svg>
            <div className="flex-1">
                <div className="text-gray-800">{chirp.user.name} <small className="text-sm text-gray-600">{chirp.created_at}</small></div>
                <div className="text-gray-800">{chirp.user.name} <small className="text-sm text-gray-600">{dayjs(chirp.created_at).fromNow()}</small></div>
                <p className="mt-4 text-lg text-gray-900">{chirp.message}</p>
            </div>
        </div>
    );
    // [tl! remove:-6,1 add:-5,1 .hidden]
}
```

Take a look in the browser to see your relative dates.

<img src="/img/screenshots/chirp-index-dates.png" alt="Chirp listing with relative dates" class="rounded-lg" />

Feel free to Chirp some more, or even register another account and start a conversation!

[Continue to allow editing of chirps](/editing-chirps)