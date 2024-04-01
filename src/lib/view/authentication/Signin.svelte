<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { createForm } from 'svelte-forms-lib';
	import {
		Button,
		Card,
		CardBody,
		CardHeader,
		Col,
		Container,
		FormGroup,
		Input,
		Label,
		Alert,
		Row
	} from 'sveltestrap';
	import * as yup from 'yup';
	import { DataService } from '../../../config/dataService';
	import Checkbox from '@components/checkbox/Checkbox.svelte';
	import { setItem, removeItem } from '../../utility/localStorageController';
	const googleIcon = '/img/svg/google-Icon.svg';

	onMount( () => {
		removeItem('user');
		removeItem('loggedin');
		removeItem('access_token');
	});

	let visible = true;
	let loader = false;
	let notification = {};

	const loginSchema = yup.object().shape({
		username: yup.string().required(),
		password: yup.string().required()
	});

	let { form, errors, handleChange, handleSubmit } = createForm({
		initialValues: {
			username: 'sakibmullas',
			password: '123'
		},
		validationSchema: loginSchema,
		onSubmit: async (values) => {
			loader = true;
			await DataService.post('/login', values)
				.then((response) => {
					console.log(response);
					if (response.data.errors) {
						console.log(response);
						notification = response.data;
					} else if (response.data.success) {
						setItem('user', response.data.data);
						setItem('loggedin', response.data.loggedin);
						setItem('access_token', response.data.data.token);
						goto(`/dashboard`);
					}
					loader = false;
				})
				.catch((error) => {
					console.log(error);
					setItem('user', '');
					setItem('loggedin', false);
					setItem('access_token', '');
					goto(`/login`);
					loader = false;
				});

			visible = true;
		}
	});
	const handleCheckBox = () => {};
</script>

<div class="main-content">
	<div class="admin">
		<Container fluid>
			<Row class="justify-content-center">
				<Col xxl="3" md="6" sm="8">
					<div class="edit-profile">
						<div class="edit-profile__logos">
							<img src="/img/hp.png" class="svg" alt="" />
						</div>
						<Card class="border-0">
							<!-- {#if loader}
								<Spinner type="primary" size="md" />
							{/if} -->
							<CardHeader>
								<div class="edit-profile__title">
									<h6>Sign in</h6>
								</div>
							</CardHeader>
							<CardBody>
								{#if notification.errors}
									<Alert
										color="danger"
										isOpen={visible}
										toggle={() => (visible = false)}
										fade={false}
									>
										{notification.errors}
									</Alert>
								{/if}
								<form  on:submit|preventDefault={handleSubmit}>
									<div class="edit-profile__body">
										<FormGroup class="form-group mb-25">
											<Label for="username">Email Address</Label>
											<Input
												on:change={handleChange}
												bind:value={$form.username}
												invalid={$errors.username.length > 0}
												type="username"
												class="form-control"
												id="username"
												name="username"
												placeholder="hexadash@dm.com"
											/>
											{#if $errors.username}
												<div class="invalid-feedback">{$errors.username}</div>
											{/if}
										</FormGroup>
										<FormGroup class="form-group mb-15">
											<Label for="password">password</Label>
											<div class="position-relative">
												<Input
													on:change={handleChange}
													bind:value={$form.password}
													invalid={$errors.password.length > 0}
													type="password"
													class="form-control"
													id="password"
													name="password"
													placeholder="Password"
												/>
												{#if $errors.password}
													<div class="invalid-feedback">{$errors.password}</div>
												{/if}
											</div>
										</FormGroup>
										<div class="admin-condition">
											<Checkbox type="default" id="check-1" onChange={handleCheckBox}>
												Keep Me Logged In
											</Checkbox>
											<a href="/forgotPass">Forget password?</a>
										</div>
										<div
											class="admin__button-group button-group d-flex pt-1 justify-content-md-start justify-content-center"
										>
											<Button
												type="submit"
												color="primary"
												class="btn btn-primary btn-default w-100 btn-squared text-capitalize lh-normal px-50 signIn-createBtn "
											>
												{loader ? 'Loading' : 'sign in'}
											</Button>
										</div>
									</div>
								</form>
							</CardBody>
						
							<div class="admin-bottom">
								<p class="mb-0">
									Don't have an account?
									<a href="/signup" class="color-primary"> Sign up </a>
								</p>
							</div>
						</Card>
					</div>
				</Col>
			</Row>
		</Container>
	</div>
</div>

<style lang="scss">
	:global {
		@import '../../../../src/assets/sass/components/style.scss';
	}
</style>
