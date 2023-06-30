
# Queue emails in laravel 10



## Installation

Install a new laravel project/ clone 

```bash
  Install a new laravel project and follow the attached steps
```

    
## Run Locally

Clone the project

```bash
  git clone this project for a ready made demo
```

Under .env add replace the mail variable with :

```bash
    MAIL_MAILER=smtp
    MAIL_HOST=smtp.office365.com
    MAIL_PORT=587
    MAIL_USERNAME=sender_email
    MAIL_PASSWORD=...
    MAIL_ENCRYPTION=tls
    MAIL_FROM_ADDRESS=sender_email
    MAIL_FROM_NAME="${APP_NAME}"
```
```bash
    QUEUE_CONNECTION=database
```

What to add 

```bash
  php artisan make:mail RegisterUser

<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class RegisterUser extends Mailable
{
    use Queueable, SerializesModels;
    public $details;
    /**
     * Create a new message instance.
     */
    public function __construct($details)
    {
        $this->details = $details;
    }

    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Register User',
        );
    }

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'view.name',
        );
    }

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [];
    }

    public function build(){
        // return $this->view('mail');
        $details = $this->details;
        return $this->markdown('mail')
        ->subject($details['name']);
    }
}

```

Make a route in Web

```bash
  Route::get('/', function(){
    $details['email'] = 'receivermail@gmail.com';
    $details['name'] = 'receivername';
    dispatch(new App\Jobs\SendEmailJob($details));
});
```
Make a mail blade under resources views:

```bash
  resources\views\mail.blade.php
```
Create a Queue table and migrate:

```bash
  php artisan queue:table
  php artisan migrate
```
Make a job:

```bash
  php artisan make:job SendEmailJob

  <?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

use App\Mail\RegisterUser;
use Illuminate\Support\Facades\Mail;

class SendEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    public $details;
    /**
     * Create a new job instance.
     */
    public function __construct($details)
    {
        $this->details = $details;
    }

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        $details = $this->details;
        $email = new RegisterUser($details);
        Mail::to($this->details['email'])->send($email);
    }
}
```
