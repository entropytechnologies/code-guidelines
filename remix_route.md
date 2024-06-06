```javascript
// Necessary imports from Remix and React
import { useLoaderData, Form, useActionData, redirect } from "@remix-run/react";
import { json } from "@remix-run/node";
import { getProfile, updateProfile } from "~/utils/profile.server";
import { requireUser } from "~/utils/session.server";

// Loader to fetch initial data
export const loader = async ({ request }) => {
  const user = await requireUser(request);
  const profile = await getProfile(user.id);
  
  return json({ profile });
};

// Action to handle form submissions. Here we can add validation logic using zod or other libraries.
export const action = async ({ request }) => {
  const formData = await request.formData();
  const name = formData.get("name");
  const email = formData.get("email");

  const user = await requireUser(request);

  try {
    await updateProfile(user.id, { name, email });
    return redirect("/profile");
  } catch (error) {
    return json({ error: error.message }, { status: 400 });
  }
};

// Profile component
export default function Profile() {
  const { profile } = useLoaderData();
  const actionData = useActionData();

  return (
    <div>
      <h1>Profile</h1>
      {actionData?.error && <p style={{ color: 'red' }}>{actionData.error}</p>}
      <Form method="post">
        <div>
          <label htmlFor="name">Name:</label>
          <input type="text" id="name" name="name" defaultValue={profile.name} />
        </div>
        <div>
          <label htmlFor="email">Email:</label>
          <input type="email" id="email" name="email" defaultValue={profile.email} />
        </div>
        <button type="submit">Update Profile</button>
      </Form>
    </div>
  );
}
